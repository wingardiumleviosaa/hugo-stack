---
title: "[Ansible] couldn't resolve module/action 'ansible.windows.win_powershell"
tags:
  - Ansible
categories:
  - SRE
date: 2022-10-23 18:52:00
slug: ansible-coundnot-resolve-module-action
---

## 問題

在 alphine 3.13 安裝 ansible 以及相關 ansible 模組後打包 docker image，在執行時 ansible.windows.win_powershell 模組時會報錯

<!--more-->

```
ERROR! couldn't resolve module/action 'ansible.windows.win_powershell'. This often indicates a misspelling, missing collection, or incorrect module path.
```

## 原因

原先使用的 docker base image 為公司內部事先包好的 alpine 3.13 版本，以該版為基礎 apk add 安裝的 ansible-core 只到 2.10，其中 ansible.windows collection 並無包含 win_powershell 模組。

## 解決方法

將 docker base image alpine 升級到 3.16，即可解決。

補充一下，在公司內部網路上無法直接使用 dockerhub 的 alpine，因為沒有 proxy 出不去外網，故須先自己設 proxy

```docker
FROM alpine:3.16

ENV http_proxy=http://whqproxys.wistron.com:8080
ENV https_proxy=http://whqproxys.wistron.com:8080
ENV no_proxy=127.0.0.1,localhost,wistron.com,wistron.com.cn

RUN apk upgrade && rm -rf /var/cache/apk/*
```

## Reference

- [https://bytemeta.vip/repo/ansible-collections/ansible.windows/issues/282](https://bytemeta.vip/repo/ansible-collections/ansible.windows/issues/282)
