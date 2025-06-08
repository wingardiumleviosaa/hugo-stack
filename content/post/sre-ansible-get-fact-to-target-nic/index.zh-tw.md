---
title: "[Ansible] get_fact ipv4 address 取到 eth0"
tags:
  - Ansible
categories:
  - SRE
date: 2022-05-19 13:02:00
slug: ansible-get-fact-to-target-nic
---

## TL; DR

Ansible 的控制節點以及受控節點皆使用 VM 打起來，在控制節點使用 get-facts 要取得受控節點的 IP 資料時，總是會取得第一張網卡 eth0，但目標是要取得第二張網卡。

<!--more-->

```
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 08:00:27:84:06:a3
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe84:6a3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1922 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1247 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:191345 (191.3 KB)  TX bytes:163962 (163.9 KB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:fc:bb:de
          inet addr:192.168.56.10  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fefc:bbde/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:132 errors:0 dropped:0 overruns:0 frame:0
          TX packets:91 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:109552 (109.5 KB)  TX bytes:23202 (23.2 KB)
```

## 解決方式

在系統的路由表中添加一個靜態路由，該路由指定將流量發送到 8.8.8.8（Google DNS 服務器）的接口為 eth1。這樣，當系統嘗試訪問 8.8.8.8 時，將使用指定的接口進行通信。

```sh
route add -net 8.8.8.8 netmask 255.255.255.255 eth1
```

## Reference

- https://superuser.com/questions/991711/vagrant-virtualbox-doesnt-setup-correct-gateway
