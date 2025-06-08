---
title: "[CentOS] Crash Analysis (kexec & kdump)"
categories: ["SRE"]
tags:
  - CentOS
date: 2022-02-08 18:58:00
slug: "centos-crash-analysis"
---

## 原理

`kexec`（kernel execution）是 Linux 核心的一種機制，允許從當前執行的核心啟動新核心。kexec 會繞過系統韌體 (BIOS or UEFI) 的初始化，並直接將新核心載入到主記憶體執行，可以實現系統的快速重啟。

<!--more-->

`kdump` 是一種基於 kexec 實現的**核心崩潰轉儲技術**。當系統崩潰時，kdump 使用 kexec 啟動另一個核心並獲得記憶體轉儲，並使用它來匯出和儲存記憶體轉儲來保持系統一致性，最終會匯出一個記憶體映像（也稱為 vmcore），該映像可用於除錯和確定崩潰的原因。

![](https://imgur.com/qVB7Gv4.png)

`crash` 是一個被廣泛應用的核心崩潰轉儲檔案分析工具，可以通過 crash 分析 vmcore 檔案以分析出核心崩潰的原因。

## 安裝

### 1. 檢視系統核心

```bash
[root@kvm2 ~]# uname  -r
3.10.0-1127.el7.x86_64
```

另外說明一下目前使用的 CentOS 版本為 7.9

```bash
[root@kvm2 ~]# cat /etc/centos-release
CentOS Linux release 7.9.2009 (Core)
```

### 2. 安裝 kexec & crash

```bash
yum install crash kexec-tools -y
```

kdump 通常在安裝 CentOS 時就會預設開啟了

![](https://imgur.com/3tNR6NA.png)

{{< notice note >}}

- 網路上有些教學會寫需要修改 grub 的 crashkernel 預留記憶體大小、以及更新 kdump.conf 配置，但也可以不配置、不更新；皆保持預設。
- 在 Linux 4.15 中預設使用 `crashkernel=auto`，kernel 將通過 memblock_find_in_range 自動計算核心的記憶體大小和起始位置，但是有些核心可能不支持，需要手動指定。
- 如果要自己手動設定，可參考 <a href="https://www.linuxtechi.com/how-to-enable-kdump-on-rhel-7-and-centos-7/">這篇教學</a>
  {{< /notice >}}

### 3. 安裝 kernel-debuginfo

使用 crash 除錯核心轉儲檔案，需要安裝 crash 工具和核心除錯工具 kernel-debuginfo。下載連結 [http://debuginfo.centos.org/7/x86_64/](http://debuginfo.centos.org/7/x86_64/)

```bash
wget http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-common-x86_64-3.10.0-1127.el7.x86_64.rpm
wget http://debuginfo.centos.org/7/x86_64/kernel-debuginfo-3.10.0-1127.el7.x86_64.rpm
rpm -ivh kernel-debuginfo-3.10.0-1160.15.2.el7.x86_64.rpm kernel-debuginfo-common-x86_64-3.10.0-1160.15.2.el7.x86_64.rpm
```

## 測試並使用 crash 分析 vmcore 報告

### 1. 測試 kdump 是否有安裝成功

要測試 kdump 是否設定成功，最準確的方法就是產生一個 kernel crash，看看 kdump 是不是會捕捉到錯誤並產生 dump 檔。執行下面的指令，讓 kernel 立刻產生一個 crash：

```bash
echo "c" > /proc/sysrq-trigger
```

### 2. 系統重啟後，查看 vmcore 檔案

進入 /var/crash 目錄，如果有看到接近現在時間的目錄，就代表剛剛刻意製造的 crash 有成功產生 kernel dump 了，目錄裡面的 vmcore 就是 kernel dump 檔：

```bash
[root@kvm2 crash]# cd /var/crash
[root@kvm2 crash]# ll
total 0
drwxr-xr-x.  4 root root  80 Feb  7 16:29 .
drwxr-xr-x. 19 root root 267 Nov 16  2020 ..
drwxr-xr-x.  2 root root  44 Feb  7 16:18 127.0.0.1-2021-04-14-03:24:39
drwxr-xr-x.  2 root root  44 Feb  7 16:29 127.0.0.1-2022-02-07-16:29:37
[root@kvm2 crash]# cd 127.0.0.1-2022-02-07-16\:29\:37/
[root@kvm2 127.0.0.1-2022-02-07-16:29:37]# ll
total 1116400
drwxr-xr-x. 2 root root         44 Feb  7 16:29 .
drwxr-xr-x. 4 root root         80 Feb  7 16:29 ..
-rw-------. 1 root root 1143071638 Feb  7 16:29 vmcore
-rw-r--r--. 1 root root     118713 Feb  7 16:29 vmcore-dmesg.txt
```

### 3. 使用 crash 命令載入 vmcore 檔案

```bash
[root@kvm2 127.0.0.1-2022-02-07-16:29:37]# crash /usr/lib/debug/lib/modules/3.10.0-1127.el7.x86_64/vmlinux vmcore

crash 7.2.3-11.el7_9.1
Copyright (C) 2002-2017  Red Hat, Inc.
Copyright (C) 2004, 2005, 2006, 2010  IBM Corporation
Copyright (C) 1999-2006  Hewlett-Packard Co
Copyright (C) 2005, 2006, 2011, 2012  Fujitsu Limited
Copyright (C) 2006, 2007  VA Linux Systems Japan K.K.
Copyright (C) 2005, 2011  NEC Corporation
Copyright (C) 1999, 2002, 2007  Silicon Graphics, Inc.
Copyright (C) 1999, 2000, 2001, 2002  Mission Critical Linux, Inc.
This program is free software, covered by the GNU General Public License,
and you are welcome to change it and/or distribute copies of it under
certain conditions.  Enter "help copying" to see the conditions.
This program has absolutely no warranty.  Enter "help warranty" for details.

GNU gdb (GDB) 7.6
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu"...

WARNING: kernel relocated [394MB]: patching 87167 gdb minimal_symbol values

      KERNEL: /usr/lib/debug/lib/modules/3.10.0-1127.el7.x86_64/vmlinux
    DUMPFILE: vmcore  [PARTIAL DUMP]
        CPUS: 48
        DATE: Mon Feb  7 16:29:31 2022
      UPTIME: 01:43:46
LOAD AVERAGE: 1.04, 1.28, 1.42
       TASKS: 657
    NODENAME: kvm2
     RELEASE: 3.10.0-1127.el7.x86_64
     VERSION: #1 SMP Tue Mar 31 23:36:51 UTC 2020
     MACHINE: x86_64  (2199 Mhz)
      MEMORY: 255.9 GB
       PANIC: "SysRq : Trigger a crash"
         PID: 16442
     COMMAND: "bash"
        TASK: ffff926f7c1bd230  [THREAD_INFO: ffff926ca58b0000]
         CPU: 10
       STATE: TASK_RUNNING (SYSRQ)

crash>

```

解釋：

- KERNEL：系統崩潰時執行的 kernel 檔案
- DUMPFILE： 核心轉儲檔案
- CPUS： 所在機器的 CPU 數量
- DATE：系統崩潰的時間
- TASKS：系統崩潰時記憶體中的任務數
- NODENAME：崩潰的系統主機名
- RELEASE: 和 VERSION：核心版本號
- MACHINE：CPU 架構
- MEMORY：崩潰主機的實體記憶體
- PANIC：崩潰型別，常見的崩潰型別包括：
- SysRq (System Request)：通過魔法組合鍵導致的系統崩潰，通常是測試使用。通過 echo c > /proc/sysrq-trigger，就可以觸發系統崩潰。
- oops：可以看成是核心級的 Segmentation Fault。應用程式如果進行了非法記憶體訪問或執行了非法指令，會得到 Segfault 訊號，一般行為是 coredump，應用程式也可以自己截獲 Segfault 訊號，自行處理。如果核心自己犯了這樣的錯誤，則會彈出 oops 資訊。

## Reference

- https://iter01.com/588761.html
- https://winddoing.github.io/post/8229.html
- https://zh.wikipedia.org/
- https://www.if-not-true-then-false.com/
