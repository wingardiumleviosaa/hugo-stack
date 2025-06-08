---
title: Ubuntu 20.04 安裝本地 deb 包
tags:
  - Ubuntu
categories:
  - SRE
date: 2022-05-09 19:54:00
slug: ubuntu-install-deb-file
---

apt 是 ubuntu 最常用的包命令，用於從 Ubuntu 存儲庫、PPA 和第三方 apt 存儲庫安裝、刪除和管理 package。從 Ubuntu 20.04 開始，apt 命令支持對本地 deb 文件的安裝。

<!--more-->

## dpkg

在以前都是使用 `dpkg -i` 來安裝本地 deb。但是 dpkg 不會自動安裝依賴包，因此安裝很容易出現依賴相關的錯誤。之後需要通過運行 `sudo apt-get install -f` 來安裝依賴。

## apt/apt-get

直接通過 apt/apt-get 來安裝本地 deb 包，只需要為 apt/apt-get 指定 deb 包的相對路徑或絕對路徑就行了，不能直接在 apt 命令後指定 deb 包的名字，必須要指定路徑，否則 apt 命令會嘗試從遠程倉庫中搜索 deb 包同名的 package，從而導致安裝失敗。如下範例：

```bash
sudo apt install ./PACKAGE_NAME.deb
sudo apt install /home/ula/PACKAGE_NAME.deb
```

## Reference

- https://ubuntuhandbook.org/index.php/2021/04/install-deb-file-ubuntu-4-ways/
