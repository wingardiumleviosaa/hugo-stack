---
title: "[Linux] LVM"
categories: ["SRE"]
tags: ["Linux"]
date: 2020-11-27 17:37:00
slug: "linux-lvm"
---

LVM (Logical Volume Manager) 邏輯磁碟管理器，是 Linux 系統 kernel 所提供的功能，在硬碟的分割區上建立邏輯層，可以彈性調整磁碟、分割區與檔案系統的大小。

<!--more-->

## LVM 層級

將實體分割區變成 Physical Volume (PV)，由 PV 組成虛擬硬碟 Virtual Group (VG)，VG 可以分割成數個虛擬分割區 Logical Volume (LV)，然後在 LV 上建立檔案系統，可以當成一般檔案系統使用。

![](https://imgur.com/Kd6bDCl.png)

## LVM 實作

- 安裝

```
$ yum install -y lvm2
```

- 列出磁碟資訊

```
$ fdisk -l
```

- 分割磁碟

```
$ fdisk /dev/sdb
```

進入對話框後，依提示作相對應的選擇。

1. n # 開始分割
2. 按 enter # default is p for primary
3. 按 enter # 預設分一區
4. 按 enter # 預設開頭可行的磁區
5. 按 enter # 預設結尾可行的磁區
6. t # 更動型態
7. 輸入 LVM 的 Hex Code 8e00
8. w # 寫入磁區設定並離開

更新 Linux Kernel 分割表資訊

```
$ partprobe
```

- 建立 Pysical Volume (PV)  
  使用 `pvcreate` 標記指定磁碟分區為要使用的 LVM PV。

```
$ pvcreate /dev/sdb1
$ pvcreate /dev/sdb1 /dev/sdc1 # 或建立多個 pv
```

- 建立 Virtual Group (VG)

```
$ vgcreate vgName /dev/sdb1

# 或多個 PV 建成一個 VG
$ vgcreate vgName /dev/sdb1 /dev/sdc1
```

- 建立 Logical Volume (LV)
  - -n (to set the LV name)
  - -L (LV Size in bytes)
  - the VG name for the LV

```
$ lvcreate -n lvName -L 100M vgName
```

查詢已建立虛擬磁區

```
$ lvdisplay
```

<br>

{{< notice info >}}
不同的工具會使用不同的 LV 路徑，傳統名稱 /dev/vgName/lvName 或 kernel device mapper 名稱 /dev/mapper/vgname-lvname。
{{< /notice >}}

- 格式化虛擬磁區  
  格式化為 xfs 文件系統

```
$ mkfs.xfs /dev/vgName/lvName
```

- 掛載  
  在目錄中創建新的 LV，再將磁區掛載上去。

```
$ mkdir data
$ mount /dev/vgName/lvName /data
```

確認一下

```
$ df -h
```

把 mount point 記錄到 fstab 檔案下

```
$ vi /etc/fstab
```

<br>

```
/dev/vgName/lvName    /data    xfs    defaults    0   0
1
```

### 把原有的 lvm 目錄增大

```
# 擴大 LV
$　lvextend -L +1T /dev/vgName/lvName
# 擴大檔案系統
$ resize2fs /dev/vgName/lvName
```

## Reference

- https://zh.wikipedia.org/wiki/%E9%82%8F%E8%BC%AF%E6%8D%B2%E8%BB%B8%E7%AE%A1%E7%90%86%E5%93%A1
- https://www.sysonion.de/centos-logical-volume-management-lvm/
- https://sc8log.blogspot.com/2017/03/linux-lvm-lvm.html
