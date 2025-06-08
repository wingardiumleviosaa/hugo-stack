---
title: "[Docker] Install Docker & Docker Compose on Ubuntu 18.04"
tags:
  - Docker
categories:
  - SRE
date: 2020-08-18 14:48:00
slug: install-docker-and-docker-compose-on-ubuntu18
---

官方 Ubuntu repository 中提供的 Docker 可能不是最新版本。 為確保獲得最新版本，本篇將從官方 Docker 存儲庫安裝。 為此，我們將添加一個新的 package source，從 Docker 中添加 GPG 密鑰以確保下載有效，然後安裝該程序包。

<!--more-->

## 安裝 Docker

### 更新 package 並匯入 docker repo

先更新現有的 packages

```
sudo apt update
```

安裝需要用到的套件

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

將官方 docker repo 的 gpg 密鑰匯入

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

將 docker 穩定版 repo 加到 apt source

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

再次更新 packages

```
sudo apt update
```

### 安裝最新版或是指定版本

- 安裝最新版

```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

- 安裝指定版本
  a. 列出 repo 中所有可用版本

```
apt-cache madison docker-ce
docker-ce | 5:19.03.13~1.2.beta2-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/test amd64 Packages
 docker-ce | 5:19.03.12~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/test amd64 Packages
 docker-ce | 5:19.03.11~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/test amd64 Packages
......
```

</br>

```
 apt-cache madison docker-ce-cli
docker-ce-cli | 5:19.03.13~1.2.beta2-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/test amd64 Packages
docker-ce-cli | 5:19.03.12~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/test amd64 Packages
......
```

b. 選擇指定版本安裝，例如 5:19.03.12~3-0~ubuntu-bionic

```
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

### 完成安裝

安裝成功後 daemon 會自動啟用，並且會加到開機自動啟動的程序中。可以使用下列命令確認安裝以及程序狀態：
確認版本資訊

```
docker version
```

![](https://imgur.com/lUWfH3W.png)

確認狀態

```
sudo systemctl status docker
```

![](https://imgur.com/speYw5F.png)

### 將當下使用者加入 docker group

默認情況下，只能以 root 或由 docker group 中的用戶執行 docker 命令。否則會得到下列警示：

```
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

如果要避免在每次運行 docker 命令時都加 sudo，請將用戶添加到 docker group：

```
sudo usermod -aG docker ${USER}
su - ${USER}
```

確認用戶目前所屬的 group

```
id -nG
```

---

上面演示了如何手動安裝 docker，有點麻煩，所以官方提供了[腳本](https://get.docker.com/)可以一鍵安裝。

## 使用官方安装腳本自動安裝

```
curl -fsSL https://get.docker.com | sudo sh
```

完成！超簡潔。

---

## 安裝 docker-compose

### 下載最新版

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

若要安裝其他版本，請將 URL 中的 `1.26.2` 取代成其他[版本](https://docs.docker.com/compose/release-notes/)。

### 把 docker-compose 加入執行檔

```
sudo chmod +x /usr/local/bin/docker-compose
```

### 測試安裝

```
docker-compose --version
docker-compose version 1.26.2, build 1110ad01
```

## reference

- https://docs.docker.com/engine/install/ubuntu/
- https://docs.docker.com/compose/install/
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
