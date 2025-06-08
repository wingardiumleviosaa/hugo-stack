---
title: Linux 吃掉了記憶體 ?!
tags:
  - Linux
categories:
  - SRE
date: 2022-03-03 19:21:00
slug: linux-eat-all-memory
---

## 前言

在觀察用於儲存 k8s 的 nfs server 時，發現記憶體的 cache/buffer 值非常高，但細看 top 卻無任何應用程式佔用記憶體空間。查了一下網路，發現是 linux 系統本身的機制。

<!--more-->

## 原因

Linux 會借用未使用的記憶體來做磁碟快取，可以讓應用載入更快並且執行更加流暢，提高 IO 性能。如果有其他應用要用到記憶體時，系統會從磁碟快取中拿走一塊被借用的記憶體，不會用到 swap。

## free 命令的記憶體空間解析

```bash
[root@nfs ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           188G        6.4G        1.0G        138M        181G        181G
Swap:          4.0G          0B        4.0G
[root@nfs ~]#
```

- total 記憶體總數
- used 已經使用的記憶體
- free 空閒的記憶體數
- shared 多個進程共享的記憶體總額
- buffer 作為 buffer cache 的緩存
- cache 作為 page cache 的緩存
- available = free + buff/cache

![](https://imgur.com/jSqUSOV.png)

上表中 something 代表的正是 free 命令中 `buffers/cached` 使用的記憶體。由於這個記憶體實際上是從作業系統的角度使用的，所以如果用戶想要使用它，那麼它可以被用戶的應用快速地回收和使用。

## buffer & cache

### buffer cache 緩衝

Buffer cache 也叫塊緩衝，是對物理磁盤上的一個磁盤塊進行的緩衝，其大小爲通常爲 1k。它是爲了緩衝寫操作然後一次性將很多改動寫入硬盤，避免頻繁寫硬盤，提高寫入效率。

### page cache 快取

page cache 頁緩衝/文件緩衝，由若干個磁盤塊組成(也即由若干個 bufferCache 組成，物理上不一定連續)，通常為 4K、在 64 位系統上爲 8k。它是爲了給讀操作提供緩衝，避免頻繁讀硬盤，提高讀取效率。

簡單說來，buffer cache 用來緩存磁盤數據、優化磁盤的 I/O；page cache 用來緩存文件數據、優化文件系統的 I/O。在有文件系統的情況下，對文件操作，那麼數據會緩存到 page cache，如果直接採用 dd 等工具對磁盤進行讀寫，那麼數據會緩存到 buffer cache。

## 如何清除 cache

**一般來說使用者是不需要去管這些 cache 何時會被清除的**，所以 Linux 也沒有專門的指令來做這件事，不過依然有提供一個 proc file system 介面 `/proc/sys/vm/drop_caches`，可以強制 kernel 清理快取。drop_caches 的值可以是 0-3 之間的數字，代表不同涵義：

0：不釋放(系統默認值)；不釋放内存，由操作系統自動管理  
1：釋放 pagecache.

```
# sync && echo 1 > /proc/sys/vm/drop_caches
```

2：釋放 dentries(目錄緩存) 和 inodes (文件元數據)

```
# sync && echo 2 > /proc/sys/vm/drop_caches
```

3：釋放所有緩存 pagecache, dentries and inodes.

```
# sync && echo 3 > /proc/sys/vm/drop_caches
```

基本上執行這些指令沒有什麼好處，所以除非你很明確知道你想幹嘛（例如為了避免 cache 影響實驗結果），否則建議不要隨便手動清除快取記憶體。另外，也**建議在做這個動作前先執行 `sync` 讓檔案寫入操作先完成，否則可能會造成意想不到的結果。**

## Reference

- https://www.linuxatemyram.com/
- https://medium.com/hungys-blog/clear-linux-memory-cache-manually-90bec95ea003
