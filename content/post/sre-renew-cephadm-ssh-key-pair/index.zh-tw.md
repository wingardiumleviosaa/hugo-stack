---
title: "[Ceph] 更新 Cephadm SSH Key Pair"
tags:
  - Ceph
categories:
  - SRE
date: 2024-05-28T11:30:01+08:00
slug: ceph-renew-cephadm-ssh-key-pair
---

## TL; DR

Ceph cluster 的建置過程是使用 root 帳號在 cephadm 節點操作的，cephadm 節點會通過 ssh 跟其它節點溝通，本文紀錄如何更新 ssh key pair。

<!--more-->

## 建立新的金鑰對

```bash
sudo su
cd ~/.ssh/
ssh-keygen -t rsa -b 4096 -f ~/.ssh/cephadm-new-key
```

## 更新 authorized_key

更新包含 cephadm 的所有 ceph 節點的 root 的 authorized_key 檔案

```bash
cat ~/.ssh/cephadm-new-key.pub
# 複製公鑰內容
vi ~/.ssh/authorized_keys
# 刪除舊的公鑰，並貼上公鑰內容
```

</br>
{{< notice info >}}
⚠️ 請注意，連 cephadm 自己的 authorized_key 也須更新，否則在下一步驟設定新的金鑰對的時候會出現 `Error EINVAL: ssh connection [root@ceph-1.sdsp-stg.com](mailto:root@ceph-1.sdsp-stg.com) failed` 的錯誤。
{{< /notice >}}

## 更新 cephadm key pair

```bash
# 刪除舊的 key
ceph cephadm clear-key
# 設定新的公私鑰
ceph cephadm set-priv-key -i /root/.ssh/cephadm-new-key
ceph cephadm set-pub-key -i /root/.ssh/cephadm-new-key.pub
```
