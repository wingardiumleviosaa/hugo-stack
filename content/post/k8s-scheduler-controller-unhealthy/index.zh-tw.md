---
title: '解決 scheduler and controller-manager unhealthy state'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2021-08-31 21:15:00
slug: kubernets-scheduler-controller-manager-unhealthy
---

## Problem
在嘗試更新 Kubernetes 時，下了下面的 command 取得目前集群的組件狀態：
```
$ kubectl get cs
```
<!--more-->

發現 controller-mamager 和 scheduler 有 unhealthy 的狀態：

```
NAME                 STATUS      MESSAGE                                                                                     ERROR
controller-manager   Unhealthy   Get http://127.0.0.1:10252/healthz: dial tcp 127.0.0.1:10252: connect: connection refused
scheduler            Unhealthy   Get http://127.0.0.1:10251/healthz: dial tcp 127.0.0.1:10251: connect: connection refused
etcd-0               Healthy     {"health":"true"
```

## Reason
這兩個 pod 的非安全端口沒有開啟，健康檢查時報錯，但是由於本身服務是正常的，只是健康檢查的端口沒啟，所以不影響正常使用。

## Solution

在所有 Master nodes 上修改下面檔案:

```bash
$ vim /etc/kubernetes/manifests/kube-scheduler.yaml
$ vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

刪掉或註釋掉 `- --port=0` (spec->containers->command) 這行

```bash
$ sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```

重啟 kubelet 服務

```bash
$ sudo systemctl restart kubelet.service
```

這時10251，10252端口就開啟了，健康檢查狀態也正常了。

```bash
[root@master1 ~]# netstat -tulpn | grep '10251\|10252'
tcp6       0      0 :::10251                :::*                    LISTEN      11863/kube-schedule
tcp6       0      0 :::10252                :::*                    LISTEN      11902/kube-controll
```

## Reference
- https://www.cnblogs.com/wuliping/p/13780147.html
