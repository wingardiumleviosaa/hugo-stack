---
title: "[Linux] LVM- 擴大根目錄 & 移除 pv 碟"
categories: ["SRE"]
tags: ["Linux"]
date: 2020-12-07 17:26:00
slug: "linux-lvm-extend-root-and-remove-pv"
---

## 前言

結果上次因為 LVM 分的根目錄空間不夠，仗著硬碟多就直接不管空間過剩的 /home，直接找一顆 sdc 掛在相同的根目錄 lv 中。後來系統打算拿一些硬碟做 RAID5，才發現我的系統碟(根目錄)用的是 sda 跟 sdc，跳過了 sdb，強迫症驅使下決定把 sdc 的 pv 移除!!

<!--more-->

---

## xfsdump

XFS 是 CentOS 7 預裝的 OS 的檔案系統，在 LVM 中 XFS 只能擴大不能縮小，所以需要利用 xfsdump 備份資料。

目前掛載情況如下，預計把根目錄減掉 5.5T；

```
[root@pc100 /]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                  95G     0   95G   0% /dev
tmpfs                     95G     0   95G   0% /dev/shm
tmpfs                     95G   11M   95G   1% /run
tmpfs                     95G     0   95G   0% /sys/fs/cgroup
/dev/mapper/centos-root   10T   53G  9.9T   1% /
/dev/sda2               1014M  187M  828M  19% /boot
/dev/sda1                200M   12M  189M   6% /boot/efi
tmpfs                     19G     0   19G   0% /run/user/0
/dev/mapper/centos-home  1.0T   33M  1.0T   1% /home
/dev/sdb1                5.5T   33M  5.5T   1% /data

```

## 安裝 xfsdump 工具

```
$ yum install xfsdump -y
```

## 備份根目錄

將根目錄備份到 `/data` 下。注意，`/data` 需與根目錄在不同硬碟，因為根目錄需重建。

```bash
[root@pc100 /]# xfsdump -f /data/root.dump /
xfsdump: using file dump (drive_simple) strategy
xfsdump: version 3.1.7 (dump format 3.0) - type ^C for status and control

 ============================= dump label dialog ==============================

please enter label for this dump session (timeout in 300 sec)
 -> root_dump  ##### 指定備份會話標籤
session label entered: "root_dump"

 --------------------------------- end dialog ---------------------------------

xfsdump: level 0 dump of pc100:/
xfsdump: dump date: Fri Dec  4 03:42:56 2020
xfsdump: session id: 4d68cf3f-ea74-46cb-bf06-dc205f92309a
xfsdump: session label: "root_dump"
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 56621816960 bytes
xfsdump: /var/lib/xfsdump/inventory created

 ============================= media label dialog =============================

please enter label for media in drive 0 (timeout in 300 sec)
 -> root
media label entered: "root" #### 指定設備標籤，就是對要備份的設備做一個描述

 --------------------------------- end dialog ---------------------------------

xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsdump: dumping non-directory files
xfsdump: ending media file
xfsdump: media file size 56380840688 bytes
xfsdump: dump size (non-dir files) : 56341771264 bytes
xfsdump: dump complete: 973 seconds elapsed
xfsdump: Dump Summary:
xfsdump:   stream 0 /data/root.dump OK (success)
xfsdump: Dump Status: SUCCESS
[root@pc100 /]#
[root@pc100 /]#
[root@pc100 /]# ll -h /data/root.dump
-rw-r--r--. 1 root root 53G Dec  4 03:59 /data/root.dump
```

## 進入 Rescue Mode

使用安裝光碟，重開機後進入 `Rescue Mode`。
![](https://imgur.com/TTuTYmD.png)
![](https://imgur.com/hD802Pc.png)
進入後按 `3` 進入 Shell。
![](https://imgur.com/y6uAHJG.png)

## 縮容

輸入 lvm 進入 lvm 界面。

```
sh-4.2# lvm    # 進入 lvm
lvm> pvs       # 依次掃描 pv, vg, lv
lvm> vgs
lvm> lvs
lvm> lvchange -ay /dev/mapper/centos-root # 啟用根目錄所在的 lv，-a 是啟用 y 是 yes，需啟用 (allocatable=YES) 才能在 shell 中使用該 lv。
lvm> quit      # 返回 shell
```

縮小根目錄的 lv。

```
sh-4.2# lvreduce -L 4T /dev/mapper/centos-root
sh-4.2# pvs    # 可以發現已經有空間釋出
sh-4.2# vgs
sh-4.2# lvs
```

開始移除 pv

```
sh-4.2# pvmove /dev/sdc1           # 需要看到 no data to move for xxx，非常重要，否則會造成數據遺失
sh-4.2# pvchange -xn /dev/sdc1     # 註銷
sh-4.2# pvdisplay                  # 可以發現 /dev/sdc1 的 allocatable 的值為 NO
sh-4.2# vgreduce centos /dev/sdc1  # 將 pv 從 vg 中刪除
sh-4.2# pvremove /dev/sdc1         # 刪除 pv
```

## 重新格式化根目錄

將 /dev/mapper/centos-root 進行 xfs 格式化，並重新掛載

```
sh-4.2# mkfs.xfs /dev/mapper/centos-root
sh-4.2# mount /dev/mapper/centos-root /
sh-4.2# xfsrestore /data/root.dump /
```

離開 shell 重新開機即可。

## Reference

- https://www.dotblogs.com.tw/I_know_why_I_am/2020/10/01/042546

---

不負責聲明；；；；

中間不小心迷失方向失敗了，又不好恢復做之前的原狀，就索性重灌了。把前三個硬碟 (sda,sdb,sdc) 都拉進系統碟，重灌結束後發現根目錄一樣分配的很少，其他空間都放到 /home 下面了。所以要乖乖地:

- 把 /home 縮小
- 刪除不要用的 pv
- 最後挪剩下的空間給根目錄
- 重建兩個目錄的檔案系統

### 備份 /home

```
$ tar -zcvf /tmp/home.tar.gz /home
```

### 卸載 /home

```
$ umount /home
```

### 減少 /dev/mapper/centos-home 的 lv 大小

```
$ lvreduce -L 100G /dev/mapper/centos-home
```

### 移除 pv

```
$ pvmove /dev/sdc1           # 需要看到 no data to move for xxx，非常重要，否則會造成數據遺失
$ pvchange -xn /dev/sdc1     # 註銷
$ pvdisplay                  # 可以發現 /dev/sdc1 的 allocatable 的值為 NO
$ vgreduce centos /dev/sdc1  # 將 pv 從 vg 中刪除
$ pvremove /dev/sdc1         # 刪除 pv
```

### 重建 file system

擴展根目錄並擴大文件系統

```
$ lvextend -l +100%FREE /dev/mapper/centos-root
$ xfs_growfs /dev/mapper/centos-root
```

重建 /home 目錄並掛載再還原

```
$ mkfs.xfs -f /dev/mapper/centos-home
$ mount /dev/mapper/centos-home /home/
$ tar -zxvf /tmp/home.tar.gz -C /home/
$ rm -rf /tmp/home.tar.gz
```

## Reference

- https://blog.csdn.net/weixin_38850930/article/details/106805131
