---
title: "[KVM] guest can't access Internet"
tags:
  - KVM
categories:
  - SRE
date: 2022-02-08 20:46:00
slug: kvm-guest-cannot-access-internet
---

## 狀況

在 CentOS 建立了幾台 KVM 虛擬機，但是在系統重啟後發現這幾台機器都無法連網了。只能與 host 互 ping，無法連網、無法 ping 同網段的其他主機、KVM guest 與 guest 間也不認得。

<!--more-->

其中一個 VM 的網路設定： （CentOS 7.9）

- Host IP: 10.1.5.130
- Guest IP: 10.1.5.141
- Host Network: Bridge (br0)
- Guest KVM Network Ineterface: eth0

Ping results:

```bash
root@host:~$ ping 10.1.5.141
PING 10.0.10.13 (10.0.10.13) 56(84) bytes of data.
64 bytes from 10.0.10.13: icmp_seq=1 ttl=64 time=0.207 ms

root@host:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=58 time=119 ms

root@guest:~$ ping 10.1.5.130
PING 10.0.10.2 (10.0.10.2) 56(84) bytes of data.
64 bytes from 10.0.10.2: icmp_seq=1 ttl=64 time=0.257 ms

root@guest:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.

--- 1.1.1.1 ping statistics ---
9 packets transmitted, 0 received, 100% packet loss, time 7999ms
```

查看雙方的網路設定皆無異常，重啟 server 或 kvm 的網卡皆無效。

```
[root@host ~]# ifconfig
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.5.130  netmask 255.255.255.0  broadcast 10.1.5.255
        inet6 fe80::3c9b:4cc9:bd11:8919  prefixlen 64  scopeid 0x20<link>
        ether 0c:c4:7a:b7:39:66  txqueuelen 1000  (Ethernet)
        RX packets 5179086  bytes 284472155 (271.2 MiB)
        RX errors 0  dropped 79272  overruns 0  frame 0
        TX packets 81432  bytes 13931170 (13.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

br-3138c45de2ae: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:c1:cf:65:cc  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:fc:40:fa:e4  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens1f0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 0c:c4:7a:b7:39:66  txqueuelen 1000  (Ethernet)
        RX packets 4745249  bytes 350711453 (334.4 MiB)
        RX errors 0  dropped 39639  overruns 0  frame 0
        TX packets 117171  bytes 12372319 (11.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 567  bytes 64847 (63.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 567  bytes 64847 (63.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vnet1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc54:ff:fe12:db6c  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:12:db:6c  txqueuelen 1000  (Ethernet)
        RX packets 2674  bytes 254695 (248.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 94305  bytes 5469690 (5.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@host ~]# nmcli con show
NAME                 UUID                                  TYPE      DEVICE
bridge-br0           a2d389db-dfb8-4fb4-b5e1-c98fa4f08f71  bridge    br0
br-3138c45de2ae      3e78cb65-cc77-4a9f-b485-a672a2631b86  bridge    br-3138c45de2ae
bridge-slave-ens1f0  de082284-5730-4648-8353-804805894e46  ethernet  ens1f0
vnet1                bb2d992a-72f9-4b29-b4b6-0553f7a46772  tun       vnet1

[root@host ~]# cat /etc/sysconfig/network-scripts/ifcfg-bridge-br0
STP=yes
BRIDGING_OPTS=priority=32768
TYPE=Bridge
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=bridge-br0
UUID=a2d389db-dfb8-4fb4-b5e1-c98fa4f08f71
DEVICE=br0
ONBOOT=yes
IPADDR=10.1.5.130
PREFIX=24
GATEWAY=10.1.5.254
DNS1=10.1.1.3

#########################################

[root@guest ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.1.5.141  netmask 255.255.255.0  broadcast 10.1.5.255
        inet6 fe80::432b:25e4:8f2:328  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:12:db:6c  txqueuelen 1000  (Ethernet)
        RX packets 786  bytes 76712 (74.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 167  bytes 16793 (16.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 32  bytes 2592 (2.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 32  bytes 2592 (2.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


[root@guest ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
HWADDR="52:54:00:12:db:6c"
NAME="eth0"
UUID="c43aa0f0-7327-4807-975b-9e1dc5e46332"
DEVICE="eth0"
ONBOOT="yes"
IPADDR="10.1.5.141"
PREFIX="24"
GATEWAY="10.1.5.254"
DNS1="10.1.1.3"
```

## 原因 & 解決方法

The problem is that **Docker (which is installed on host machine) changes the default policy for the FORWARD chain in iptables to DROP.** :expressionless:

To fix the issue, a rule to allow traffic has to be added. Running this command added the required rule:

```bash
# from host
iptables -I FORWARD -i br0 -o br0 -j ACCEPT
```

## Reference

- https://askubuntu.com/questions/1134115/kvm-guest-cant-access-internet
- https://www.reddit.com/r/linuxadmin/comments/bdy6sz/kvm_guest_cant_access_internet/
