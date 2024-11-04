---
title: 'Minikube Pull Image from Private Repository in WSL2'
tags:
  - Kubernetes
  - WSL
categories:
  - Kubernetes
date: 2022-10-23 19:07:00
slug: minikube-pull-harbor-image
---
## TL;DR
在 windows 環境的 wsl2 Ubuntu 上跑了 Minikube 要用來測試 kubernetes 的應用佈屬，但一直卡在拉取公司內部 harbor 鏡像的時候報 x509: certificate signed by unknown authority 錯誤。

<!--more-->

嘗試在 docker desktop 的 docker engine 加上 insecure-registries 的參數但是 minikube 不曉得為什麼吃不到，也試過把憑證放到 /etc/ssl/certs 下但一樣不認得 =__=? 總之最後才發現 minikube 的 docker 跟 docker desktop 的 docker 環境是分開的。

![](https://imgur.com/TUWqCPL.png)

## Solution
```bash
vi ~/.minikube/machines/<PROFILE_NAME>/config.json (in my case ~/.minikube/machines/minikube/config.json)
```
add private repo on InsecureRegistry attribute (json path: HostOptions.EngineOptions.InsecureRegistry)
![](https://imgur.com/o7eb0lf.png)

```bash
minikube stop
minikube start
```

Then, change the Docker daemon from Minikube
```
eval $(minikube docker-env)
```

## Reference
- https://stackoverflow.com/questions/38748717/can-not-pull-docker-image-from-private-repo-when-using-minikube
- https://tachingchen.com/tw/blog/build-docker-image-in-minikube-vm/
- https://stackoverflow.com/questions/52310599/what-does-minikube-docker-env-mean