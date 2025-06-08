---
title: "[Docker] 在不同電腦中傳 Docker image"
tags:
  - Docker
categories:
  - SRE
date: 2020-08-18 14:54:00
slug: copy-docker-image-to-other-host
---

通常 docker image 都會放在公開的 repository DockerHub 或是私有的 docker registry 上供使用者 pull，但如果沒有打算公開到網路上或是無架設 repo 的需求，又要在別台電腦上使用 build 好的 docker image 時，就需要使用備份的方式傳遞 image。

<!--more-->

## 儲存並壓縮 image

```
$ docker save -o filename.tar dockerImage
```

- o: output，輸出檔案

![](https://imgur.com/7ybCElS.png)

## 傳到別台電腦上

```
$ pscp -scp filename.tar <username>@<ip>:<path>
```

- scp：use scp (secure copy) protocol

## 在目的地將檔案 load 到 docker 中

```
$ docker load -i filename.tar
```

- i: import，輸入檔案

![](https://imgur.com/ui0nEhc.png)

## Reference

- https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-using-a-repository
