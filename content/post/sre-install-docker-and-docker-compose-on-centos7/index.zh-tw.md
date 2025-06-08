---
title: "[Docker] Install Docker & Docker Compose on CentOS7"
tags:
  - Docker
categories:
  - SRE
date: 2020-12-15 21:52:00
slug: install-docker-and-docker-compose-on-centos7
---

## 移除舊版

```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

<!--more-->

## 透過 repository 安裝 Docker

### 設定 repo

```sh
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### 安裝 Docker engine

安裝最新版

```sh
sudo yum install docker-ce docker-ce-cli containerd.io
```

第一次安裝 Docker 的時候，會匯入 GPG 的金鑰，Docker CE 版的金鑰指紋是 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，確認無誤就選擇 y 匯入。

### 啟動 Docker

```sh
sudo systemctl start docker
```

查看是否成功跑起來

```sh
[root@test ~]# docker version
Client: Docker Engine - Community
 Version:           20.10.0
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        7287ab3
 Built:             Tue Dec  8 18:57:35 2020
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.0
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       eeddea2
  Built:            Tue Dec  8 18:56:55 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### 以非 root 使用者執行 docker

如果想要使用一般使用者帳號來執行 Docker，要先將該帳號加入 docker 群組：

```sh
sudo usermod -aG docker <USERNAME>
```

---

## 安裝 docker compose

### 使用 curl

```sh
curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose -v
```

### 使用 pytion-pip

```sh
yum -y install -y epel-release    # 安裝pip需要先安裝epel-release包
yum install -y python-pip         # 安裝pip
pip install --upgrade pip         # 升級pip
pip install docker-compose        # 安裝docker-compose
docker-compose -v                 # 查看docker-compose的版本
pip install --upgrade backports.ssl_match_hostname  # 如果安裝時報錯則下
```

## Reference

- https://docs.docker.com/engine/install/centos/
- https://www.itread01.com/content/1555398180.html
