---
title: "[Linux] RAID 介紹及實作 (HW RAID & SW RAID)"
categories: ["SRE"]
tags: ["Linux"]
date: 2020-12-07 17:40:00
slug: "linux-raid"
---

RAID（Redundant Array of Independent Disks) 磁碟陣列，多個硬碟組合成為一個邏輯磁區(作業系統只會把它當作一個硬碟)。

![](https://imgur.com/rJFbWVv.png)

RAID 一般分類成 Software RAID 與 Hardware RAID。

<!--more-->

---

## Hardware RAID

Hardware RAID 用專屬 RAID 運算晶片，晶片位於介面卡或直接嵌入在主機板上。作業系統只需要驅動 RAID 晶片即可運作，RAID 任務分工是由 RAID 晶片做，較不影響 CPU。

## ATA(SATA) RAID / BIOS RAID / Fake RAID

這類型 RAID 介於 Software RAID 與 Hardware RAID 之間，算是半個 Hardware RAID，通常由主機板晶片來「幫忙」RAID 運算，因此有 Fake RAID（假 RAID）之稱。
要啟動這個 RAID 功能，要先在 BIOS 畫面設定以後，再使作業系統能夠正確辨識裝置即可。因為這款 RAID 要在 BIOS 內開啟，所以亦有人稱之為 BIOS RAID。

## SATA RAID 設定

若不確定系統是否內建 RAID card，可以使用下面 command 確認：

```
$ lspci -vv | grep -i raid
00:1f.2 RAID bus controller: Intel Corporation 631xESB/632xESB SATA RAID Controller (rev 09)
```

### 步驟概覽

- 在 BIOS 中啟用 RAID。
- 進入 RAID configuration utility。
- 創建 RAID。

#### 進入 BIOS

進入 BIOS ，進入 `Advance` > `SATA Configuration`，將 `Configure SATA as` 從 AHCI 更改為 `RAID`。其中要確認系統碟是否存在在 SATA，要確保 `SATA/sSATA RAID Boot Select` 的選項為系統碟所在的地方，或是直接選 `Both`。
保存設置，並離開 BIOS。

#### 進入 RAID Configuration Utility

RAID configuration utility 將在 BIOS 發布之前出現。畫面應該會先出現 RAID Volumes 以及 HDD 的列表。在該步驟中，按 `CTRL-I` 進入 RAID Configuration Utility。此工具能夠創建 RAID，修改 RAID（如果已創建）、刪除 RAID 或是將 HDD 還原回非 RAID。

#### 開始創建

選擇 `CREATE VOLUME MENU` ，鍵入 Volume Name、選擇 RAID 的類型（取決於選擇的 HDD 的數量）、選擇要使用的 HDD（從 2 個到全部，如果有更多的話），和最終大小。一旦完成創建 RAID 並重新引導，便完成了。

#### 進入系統

在設完 RAID 的後，通常開機都無法直接進入作業系統反而是進入 grub rescue。
建完 hw raid 後進入系統
lsblk 便可以看到建好的 raid (ex. /dev/md126)
格式化 raid
mkfs.ext4 /dev/md126
查看 blkid
寫入 /etc/fstab
建立 mountpoint /data
mount -av
df -h

---

## Software RAID

Software RAID 是由作業系統來提供 RAID 功能，會耗用較多 CPU 運算資源，所以要運行最好是雙 CPU 以上且高時脈等級主機，不然當遇到大量運算時，整體效能會大打折扣。Linux 上 Software RAID 技術已是相當成熟，使用裝置 /dev/md0、/dev/md1、/dev/md2 依此類推，MD 表示 Mutiple Device。

## Software RAID 設定

### 步驟概覽

- 準備並分割好兩顆硬碟
- 安裝 mdadm utility
- 使用 mdadm 建立 RAID array
- 為 RAID 設備創建文件系統

#### 準備並分割好兩顆硬碟

請注意需將 partition type 設置為 `Linux raid autodetect`。
p.s. 硬碟分割之前的[文章](https://ulahsieh.netlify.app/p/linux-disk-format-and-mount/)就有記錄過了，這邊就不贅述。

```
$ sudo fdisk -l
Disk /dev/sda: 120.0 GB, 120034123776 bytes
139 heads, 49 sectors/track, 34421 cylinders, total 234441648 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x2d0f2eb3

Device Boot      Start         End      Blocks   Id  System
/dev/sda1            2048    20973567    10485760   fd  Linux raid autodetect


Disk /dev/sdb: 120.0 GB, 120034123776 bytes
139 heads, 49 sectors/track, 34421 cylinders, total 234441648 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xe69ef1f5

Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    20973567    10485760   fd  Linux raid autodetect

Disk /dev/sdc: 120.0 GB, 120034123776 bytes
139 heads, 49 sectors/track, 34421 cylinders, total 234441648 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xe69ef1f5

Device Boot      Start         End      Blocks   Id  System
/dev/sdc1            2048    20973567    10485760   fd  Linux raid autodetect
```

#### 安裝 mdadm utility

```
Ubuntu
$ sudo apt install mdadm

CentOS
$ sudo yum install mdadm
```

#### 使用 mdadm 建立 RAID array

檢查硬碟是否有現有的 RAID 在使用

```
$ sudo madam --examine /dev/sdb /dev/sdc
$ sudo madam --examine /dev/sdb1 /dev/sdc1
```

建立 RAID1 邏輯磁區，名為 `/dev/md0`。

```
$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

<br>

{{< notice info >}}
如果出現 "Device or resource busy"，則需要重開機。
{{< /notice >}}

確認結果

```
$ cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdb1[1] sdc1[0]
      10484664 blocks super 1.2 [2/2] [UU]
      [========>............]  resync = 42.3% (4440832/10484664) finish=0.4min speed=201856K/sec
```

檢查陣列 -D(--detail)，查詢指定 MD 詳細狀態訊息

```
$ sudo mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Tue Dec 1 17:49:50 2020
     Raid Level : raid1
     Array Size : 10484664 (10.00 GiB 10.74 GB)
  Used Dev Size : 10484664 (10.00 GiB 10.74 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Tue Dec 1 17:49:50 2020
          State : active, resyncing
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

 Rebuild Status : 62% complete

           Name : localhost.localdomain:0  (local to host localhost.localdomain)
           UUID : 3a8605c3:bf0bc5b3:823c9212:7b935117
         Events : 11

    Number   Major   Minor   RaidDevice State
       0       8        1        0      active sync   /dev/sdb1
       1       8       17        1      active sync   /dev/sdc1

```

檢查

```
$ sudo mdadm -E /dev/sdb1
/dev/sdb1:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 3a8605c3:bf0bc5b3:823c9212:7b935117
           Name : localhost.localdomain:0  (local to host localhost.localdomain)
  Creation Time : Tue Dec 1 17:49:50 2020
     Raid Level : raid1
   Raid Devices : 2

 Avail Dev Size : 20969472 (10.00 GiB 10.74 GB)
     Array Size : 20969328 (10.00 GiB 10.74 GB)
  Used Dev Size : 20969328 (10.00 GiB 10.74 GB)
    Data Offset : 2048 sectors
   Super Offset : 8 sectors
          State : active
    Device UUID : 10384215:18a75991:4f09b97b:1960b8cd

    Update Time : Tue Dec 1 17:49:50 2020
       Checksum : ea435554 - correct
         Events : 18


   Device Role : Active device 0
   Array State : AA ('A' == active, '.' == missing)
```

#### 為 RAID 設備創建文件系統

為 /dev/md0 創建 ext4 文件系統，並掛載到 /mnt/raid1 下。

```
$ sudo mkfs.ext4 /dev/md0
$ sudo mkdir /mnt/raid1
$ mount /dev/md0 /mnt/raid1/
$ df -h # 查看是否掛載成功
```

添加到 /etc/fstab 文件中。

```
$ sudo vim /etc/fstab
# 加入 /dev/md0 /mnt/raid1 ext4 deaults 0 0
```

使用 mount -a 來檢查 fstab 的條目是否有誤。

```
$ mount -av
```

#### 編輯 mdadm.conf 設定檔

不同作業系統存放的位置不同，

```
/etc/mdadm.conf         # Centos 7
/etc/mdadm/mdadm.conf   # Ubuntu / Debian
```

在檔案中保存 RAID 設定，

```
$ sudo mdadm --detail --scan --verbose > /etc/mdadm.conf
$ cat /etc/mdadm.conf
ARRAY /dev/md0 level=raid1 num-devices=2 metadata=1.2 spares=1 name=localhost.localdomain:0 UUID=c7a2743d:f1e0d872:b2ad29cd:e2bee48c
      devices=/dev/sdb1,/dev/sdc1
```

更新 initramfs 使 mdadm 配置保存在啟動配置過程中。

```
$ sudo update-initramfs -u
```

#### 刪除 RAID Array

刪除邏輯磁區

```
$ sudo mdadm --stop /dev/md0
mdadm: stopped /dev/md0
```

刪除 superblock，刪除後就 partition 就可以用來做新的 RAID。

```
$ sudo mdadm --zero-superblock /dev/sdb1
$ sudo mdadm --zero-superblock /dev/sdc1
```

**並註解掉原本在 `mdm.conf` 以及 `/etc/fstab` 的 md 資料。**

## Reference

- https://zh.m.wikipedia.org/zh-tw/RAID
- https://superuser.com/questions/580717/how-do-i-set-up-hardware-raid
- https://support.us.ovhcloud.com/hc/en-us/articles/360004809700-How-to-Configure-RAID-from-the-BIOS
- https://registerboy.pixnet.net/blog/post/16190824-raid-
- https://www.thomas-krenn.com/en/wiki/Linux_Software_RAID_Information
- https://www.linuxbabe.com/linux-server/linux-software-raid-1-setup
