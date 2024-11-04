---
title: '[Ingress] 指定了 TLS 憑證，卻吃到不正確的'
tags:
  - Kubernetes
  - Ingress
  - SSL
categories:
  - Kubernetes
date: 2023-07-23T17:45:00+08:00
---
## TL; DR
在公司部屬 Ingress 資源後，發現一直沒法法吃到指定的憑證，結果才發現是因為 wildcard 的問題。

<!--more-->

## Problem
公司 Kubernetes 環境的 ingress gateway 有預設的 tls 憑證 (*.southeastasia.azure.wistron.com)，我自己申請的憑證為 `*.wistron.com`，想要指定給 tls 路徑為 `abc.southeastasia.azure.wistron.com` 的網站，但部署後連網頁發現吃的憑證會是預設的，而不是指定的。

## Reason
使用 wildcard 的憑證，只能吃到第一階的域名，比如說如果有 `*.example.com` 的 SSL 證書，那麼僅適用於 `www.example.com` 或 `XXXX.example.com` 等主機，不適用於 `demo.app1.example.com` 等主機。

## Reference
- https://stackoverflow.com/questions/26744696/ssl-multilevel-subdomain-wildcard