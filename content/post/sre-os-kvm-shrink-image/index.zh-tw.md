---
title: "[KVM] 為 image 瘦身"
author: Ula
tags:
  - KVM
categories:
  - SRE
date: 2022-02-22 22:22:00
slug: kvm-shrink-image
---

## 問題說明

在建立 vm 後，發現其使用的 qcow2 image 檔案大小超級大。使用 qemu-img info 查看 vm 真正的使用容量僅 2G 但實際上卻佔用了啟動時劃分的 disk size 如範例的 60G。

<!--more-->

![](https://imgur.com/cxEZUrX.png)

```
[root@nexdata images]# qemu-img info rockym
image: rockym
file format: qcow2
virtual size: 60G (64424509440 bytes)
disk size: 2.1G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
```

## 解決方式

透過 convert 轉換 qcow2 檔案縮小：

```
qemu-img convert -p -f qcow2 ./vm-disk-original.qcow2 -O qcow2 ./vm-disk-shrinked.qcow2
```

![](https://imgur.com/EITaak4.png)

{{< notice warning >}}
以防萬一，請先將 VM 關機後再操作。
{{< /notice >}}

## Reference

- https://serverfault.com/questions/881595/kvm-guest-qcow2-larger-than-disk-size
