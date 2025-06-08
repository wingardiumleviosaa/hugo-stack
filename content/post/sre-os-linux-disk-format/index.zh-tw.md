---
title: "[Linux] 分割 & 格式化 & 掛載硬碟"
categories: ["SRE"]
tags: ["Linux"]
date: 2020-10-15 10:14:44
slug: "linux-disk-format-and-mount"
---

新硬碟放入機器後要做三件事情，分割、格式化、掛載，才可以開始儲存資料；

<!--more-->

## 查看硬碟狀態

查看目前電腦中的硬碟

```
$ root@srv1:/# ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sda2  /dev/sdb  /dev/sdc
```

列出目前系統硬碟掛載狀態

```
$ df -h
```

</br>

```
root@srv1:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev             32G     0   32G   0% /dev
tmpfs           6.3G  2.2M  6.3G   1% /run
/dev/sda2       458G   36G  399G   9% /
tmpfs            32G     0   32G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            32G     0   32G   0% /sys/fs/cgroup
tmpfs           6.3G     0  6.3G   0% /run/user/1000
```

## 硬碟分割類型

硬碟分割有兩種模式：

1. 傳統 BIOS/MBR
2. 新型 UEFI/GPT (目前有圖形介面的 BIOS 皆屬之)

### BIOS/MBR

主開機紀錄（Master Boot Record，MBR）是電腦開機後存取硬碟時首先會讀取的第一個磁區。
開機過程是 BIOS → MBR → 啟動 OS。

### UEFI/GPT

統一可延伸韌體介面（Unified Extensible Firmware Interface，UEFI），傳統 BIOS 的改良，GPT（GUID Partition Table）就是取代 MBR 的新的磁碟分割表。
開機過程是 UEFI → GPT → 啟動 OS。

## 進入正題

### 分割

#### 小於 2TB 硬碟用 MBR

使用 `fdisk` 分割，後面接要分割的硬碟。進入後下 `m` 列出指令簡介；

```sh
root@srv1:/# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x587447ca.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
```

使用 `p` 印出目前已被分隔的硬碟，使用 `n` 開始分割，因為只要分割一個，所以一直按 enter 保持預設選項。

```
Command (m for help): p
Disk /dev/sdb: 465.78 GiB, 500107862016 bytes, 976773168 sectors
Disk model: ST3500418AS
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x587447ca

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-976773167, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-976773167, default 976773167):

Created a new partition 1 of type 'Linux' and of size 465.8 GiB.
```

最後記得下 `w` 使分割生效。

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

#### 大於 2TB 硬碟用 GPT

因為手邊沒有大於 2T 的硬碟，這邊列出進入的畫面以及用法，大致上跟 fdisk 差不多。

```
root@srv1:/# gdisk /dev/sdb
GPT fdisk (gdisk) version 1.0.5

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): ?
b       back up GPT data to a file
c       change a partition's name
d       delete a partition
i       show detailed information on a partition
l       list known partition types
n       add a new partition
o       create a new empty GUID partition table (GPT)
p       print the partition table
q       quit without saving changes
r       recovery and transformation options (experts only)
s       sort partitions
t       change a partition's type code
v       verify disk
w       write table to disk and exit
x       extra functionality (experts only)
?       print this menu
```

### 格式化

格式化意即建置檔案系統 (make filesystem)，使用 mkfs 指令，按下 tab 鍵後可以發現有許多可以指定的檔案系統，最常用的是 ext4 以及 xfs。

```
root@srv1:/# mkfs
mkfs         mkfs.btrfs   mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.ntfs    mkfs.xfs
mkfs.bfs     mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.vfat
```

以下將使用 ext4 來格式化

```
root@srv1:/# mkfs.ext4 [option] 硬碟名稱
選項與參數：
-b  ：設定 block 的大小，有 1K, 2K, 4K 的容量。
-I  ：inode-size，指定 inode 大小。
等等的還有很多參數，默認配置文件在 /etc/mke2fs.conf 設置。
```

</br>

```
root@srv1:/# mkfs.ext4 /dev/sdb1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 122096385 4k blocks and 30531584 inodes
Filesystem UUID: 06b9962b-ce9e-486c-9038-110d89564419
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done

root@srv1:/#
```

### 掛載硬碟

在 Linux 下面的磁碟掛載設定都是寫在 `/etc/fstab` 中，先看一下裡面內容；

```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/2106ed6e-8db6-4a23-af57-02accf7f2bbb / ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
```

用 `blkid` 指令列出所有磁碟的 UUID：

```
root@nexmasa-srv1:/# blkid
/dev/sda1: PARTUUID="ffba582c-4540-40b0-8881-8262b0fe846a"
/dev/sda2: UUID="2106ed6e-8db6-4a23-af57-02accf7f2bbb" TYPE="ext4" PARTUUID="a4c6e542-de4e-41ad-94c1-8cfcc3fc743b"
/dev/sdb1: UUID="fb78cf8d-9b15-408d-b005-ee8eb60231a7" TYPE="ext4" PARTUUID="cbcd5e7c-01"
root@nexmasa-srv1:/#
```

把欲掛載硬碟的 UUID 記起來，並寫到 `/etc/fstab` 中：

```
UUID=fb78cf8d-9b15-408d-b005-ee8eb60231a7 /mnt/newdisk ext4 defaults 0 0
```

**並重新開機，就大功告成了!**

```
root@srv1:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev             32G     0   32G   0% /dev
tmpfs           6.3G  2.2M  6.3G   1% /run
/dev/sda2       458G   36G  399G   9% /
tmpfs            32G     0   32G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            32G     0   32G   0% /sys/fs/cgroup
tmpfs           6.3G     0  6.3G   0% /run/user/1000
/dev/sdb1       458G   73M  435G   1% /mnt/newdisk
```

## Reference

- https://www.netadmin.com.tw/netadmin/zh-tw/technology/23D09E63D4CD46349410CDA0E36FC465
- https://www.claytontan.net/2020/01/10/%E7%A3%81%E7%A2%9F%E5%88%86%E5%89%B2-mbr-vs-gpt/
- https://blog.gtwang.org/linux/linux-add-format-mount-harddisk/
