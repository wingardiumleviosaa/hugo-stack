---
title: "[Docker] 在 wsl docker 出現 exec no such file or directory 錯誤"
tags:
  - Docker
  - WSL
categories:
  - DevOps
date: 2022-12-28 20:22:00
slug: docker-exec-no-such-file-or-directory-in-wsl
---

## TL;DR

在 windows 環境下 build docker image 並跑 container 時會出現 `exec xxx no such file or directory` 的錯誤，但明明使用同樣的 dockerfile 在 linux 環境卻沒問題。

<!--more-->

![](https://imgur.com/g2HMXuT.png)

## Reason

原因是因為換行問題，Docker 不認識 Windows 的 CRLF 換行。可以看這個範例更清楚，在 alpine container 環境原本的 CRLF 換行會被解譯成 `\r`。

![](https://imgur.com/D6NVpk0.png)

## Solution

將 Dockerfile 的換行符從 CRLF 改成 LF，可使用 VS Code 最下方 status bar 的 Select End of Line Sequence 功能，並存檔再重 build 即可。

![](https://imgur.com/gu7HaTp.png)
