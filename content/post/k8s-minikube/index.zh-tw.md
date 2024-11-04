---
title: 'Minikube'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2021-09-05 22:17:00
slug: kubernets-minikube
---

minikube 是一個由 Google 發布的部署單節點的 Kubernetes Cluster 的工具，可以安裝在本機上，支援 Windows 與 Mac Minikube 只有一個 Node (節點)。對於本地實驗可以避免節點不足的困擾；讓開發者可以在本機上輕易架設一個 Kubernetes Cluster，快速上手 Kubernetes 的指令與環境。
<!--more-->
運作原理就是會在本機上建立一個 virtual machine，並且在這 VM 建立一個 signle-node Kubernetes Cluster。

minikube 適合用於開發環境測試，不會把它用在實際生產環境中。

## 下載與部屬

Minikube 支援 Windows、MacOS、Linux，在這三種平台的本機端都可以安裝並執行 Minikube 。安裝及執行步驟，請參考[官網](https://minikube.sigs.k8s.io/docs/start/)。

整體步驟如下：

- 安裝Virtualization Software，如 [VirtualBox](http://virtualbox.org/)
- 安裝 [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) 套件，用以和 K8S 集群交互溝通
- 從 [Github](https://github.com/kubernetes/minikube) 下載 Minikube 套件
- 啟動 minikube 及 K8s 集群
- 使用 kubectl 操作集群及應用

官網跟其他教學文寫得很詳細，在這裡就不一一列示了。

## Reference

- [https://ithelp.ithome.com.tw/articles/10192490](https://ithelp.ithome.com.tw/articles/10192490)