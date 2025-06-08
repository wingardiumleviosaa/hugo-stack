---
title: "[KVM] Install Rocky Linux"
tags:
  - KVM
categories:
  - SRE
date: 2022-02-23 16:37:00
slug: kvm-install-rocky-linux
---

## 安裝 KVM 套件

請參考之前的[文章](https://ulahsieh.netlify.app/p/kvm-virt-install-centos-vm/)

<!--more-->

## 安裝 Rocky Linux VM

### 於官網下載 iso 映像檔

```
wget --no-check-certificate https://download.rockylinux.org/pub/rocky/8/isos/x86_64/Rocky-8.5-x86_64-minimal.iso
```

### 開始安裝

```
virt-install --name rockym \
--disk path=/var/lib/libvirt/images/rockym,size=60,format=qcow2 \
--vcpus 4 --memory 16384 \
--network bridge=br0 \
--graphics none --os-type linux --os-variant=rhl8.0 \
--location /var/lib/libvirt/images/Rocky-8.5-x86_64-minimal.iso \
--extra-args 'console=ttyS0'
```

### 3) Installation source 選項配置

```
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [x] Installation source               4) [x] Software selection
       (Local media)                            (Server)
5) [!] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)

Please make a selection from the above ['b' to begin installation, 'q' to quit,
'r' to refresh]: 3
================================================================================
================================================================================
Installation source

Choose an installation source type.
1) CD/DVD
2) local ISO file
3) Network

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: 2
================================================================================
================================================================================
Select device containing the ISO file

No mountable devices found

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: c
================================================================================
================================================================================
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [!] Installation source               4) [!] Software selection
       (Processing...)                          (Processing...)
5) [!] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)

Please make a selection from the above ['b' to begin installation, 'q' to quit,
'r' to refresh]: r
================================================================================
================================================================================
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [x] Installation source               4) [!] Software selection
       (Local media)                            (Source changed - please verify)
5) [!] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)
```

### 4) Software selection 選項配置

```
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [x] Installation source               4) [!] Software selection
       (Local media)                            (Source changed - please verify)
5) [x] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)

Please make a selection from the above ['b' to begin installation, 'q' to quit,
'r' to refresh]: 4
================================================================================
================================================================================
Software selection

Base environment

1) [ ] Server                           3) [ ] Custom Operating System
2) [ ] Minimal Install

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: 2
================================================================================
================================================================================
Software selection

Base environment

1) [ ] Server                           3) [ ] Custom Operating System
2) [x] Minimal Install

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: c
================================================================================
================================================================================
Software selection

Additional software for selected environment

1) [ ] Standard                         6) [ ] Network Servers
2) [ ] Development Tools                7) [ ] Scientific Support
3) [ ] Graphical Administration Tools   8) [ ] Security Tools
4) [ ] Headless Management              9) [ ] Smart Card Support
5) [ ] Legacy UNIX Compatibility        10) [ ] System Tools

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: c
================================================================================
================================================================================
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [!] Installation source               4) [!] Software selection
       (Processing...)                          (Processing...)
5) [x] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)

Please make a selection from the above ['b' to begin installation, 'q' to quit,
'r' to refresh]: r
================================================================================
================================================================================
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [x] Installation source               4) [x] Software selection
       (Local media)                            (Minimal Install)
5) [x] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)
```

其中 addtional software 的安裝選項的解釋如下:

- Standard: The standard installation of Rocky Linux.
- Development Tools: A basic development environment.
- Graphical Administration Tools: Graphical system administration tools for managing many aspects of a system.
- Headless Management: Tools for managing the system without an attached graphical console.
- Lagacy UNIX Compatibility: Compatibility programs for migration from or working with lagacy UNIX environments.
- Network Servers: These packages include network-based servers such as DHCP, Kerberos and NIS.
- Scientific Support: Tools for mathematical and scientific computations, and parallel computing.
- Security Tools: Security tools for integrity and trust verification.
- Smart Card Support: Support for using smart card authentication.
- System Tools: This group is a collection of various tools for the system, such as the client for connection to SMB shares and tools to monitor network traffic.

### 5) Installation Destination 選項配置

```
Installation

1) [x] Language settings                 2) [x] Time settings
       (English (United States))                (Asia/Taipei timezone)
3) [x] Installation source               4) [!] Software selection
       (Local media)                            (Source changed - please verify)
5) [!] Installation Destination          6) [x] Kdump
       (Automatic partitioning                  (Kdump is enabled)
       selected)
7) [ ] Network configuration             8) [!] Root password
       (Not connected)                          (Root account is disabled.)
9) [!] User creation
       (No user will be created)

Please make a selection from the above ['b' to begin installation, 'q' to quit,
'r' to refresh]: 5
Probing storage...
================================================================================
================================================================================
Installation Destination

1) [x] QEMU HARDDISK: 60 GiB (sda)

1 disk selected; 60 GiB capacity; 60 GiB free

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: c
================================================================================
================================================================================
Partitioning Options

1) [ ] Replace Existing Linux system(s)
2) [x] Use All Space
3) [ ] Use Free Space
4) [ ] Manually assign mount points

Installation requires partitioning of your hard drive. Select what space to use
for the install target or manually assign mount points.

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]: c
================================================================================
================================================================================
Partition Scheme Options

1) [ ] Standard Partition # 標準分割槽
2) [x] LVM # 邏輯卷管理
3) [ ] LVM Thin Provisioning # LVM 精簡卷

Select a partition scheme configuration.

Please make a selection from the above ['c' to continue, 'q' to quit, 'r' to
refresh]:
```

## 網路設定

就會重新開機進入登入視窗，使用剛剛設置的 root 登入。

![](https://imgur.com/zH26gR0.png)

### 設置網卡

```bash
[root@rockym ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:aa:7c:2e brd ff:ff:ff:ff:ff:ff
[root@rockym ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens2
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=ens2
UUID=f835fd27-3163-4160-9b89-387189ac11d7
DEVICE=ens2
ONBOOT=yes
IPADDR=10.1.5.143
PREFIX=24
GATEWAY=10.1.5.254
DNS1=10.1.1.3
~
~
~
~
~
"/etc/sysconfig/network-scripts/ifcfg-ens2" 18L, 307C written
[root@rockym ~]#
```

### 重啟網路

```bash
[root@rockym ~]# systemctl restart NetworkManager
[root@rockym ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:aa:7c:2e brd ff:ff:ff:ff:ff:ff
    inet 10.1.5.143/24 brd 10.1.5.255 scope global noprefixroute ens2
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:feaa:7c2e/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
[root@rockym ~]# ping www.google.com
PING www.google.com (172.217.163.36) 56(84) bytes of data.
64 bytes from tsa01s13-in-f4.1e100.net (172.217.163.36): icmp_seq=1 ttl=115 time=4.99 ms
64 bytes from tsa01s13-in-f4.1e100.net (172.217.163.36): icmp_seq=2 ttl=115 time=27.5 ms

--- www.google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 4.994/16.258/27.522/11.264 ms
[root@rockym ~]#
```

## Reference

- https://docs.rockylinux.org/guides/installation/
- https://techviewleo.com/install-kvm-with-virtualization-manager-on-rocky-linux/
