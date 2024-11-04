---
title: 使用 helm 安裝 Metrics Server
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-03-21 21:28:00
slug: kubernets-install-metrics-server-by-helm
---

Metrics Server 通過 kubelet（cAdvisor）獲取監控數據，主要作用是為 kube-scheduler、HPA(Horizontal Pod Autoscaler)等 k8s 核心組件，以及 kubectl top 命令和 Dashboard 等 UI 組件提供數據來源，可以用來看 node 或 pod 的資源 (CPU & Memory) 消耗。須注意的是，Metric Server 是 in memory 的 monitor，只可以查詢當前的度量數據，並不保存歷史數據。

<!--more-->

## 安裝
```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server --namespace kube-system
Release "metrics-server" does not exist. Installing it now.
NAME: metrics-server
LAST DEPLOYED: Mon Mar 21 16:32:14 2022
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
***********************************************************************
* Metrics Server                                                      *
***********************************************************************
  Chart version: 3.8.2
  App version:   0.6.1
  Image tag:     k8s.gcr.io/metrics-server/metrics-server:v0.6.1
***********************************************************************
```

## 連線失敗問題
可以發現使用預設值安裝完後 deployment 一直無法 ready，查看 deployment 資訊
```yaml
# kubectl describe deployments.apps -n kube-system metrics-server
Name:                   metrics-server
Namespace:              kube-system
CreationTimestamp:      Mon, 21 Mar 2022 16:32:17 +0800
Labels:                 app.kubernetes.io/instance=metrics-server
                        app.kubernetes.io/managed-by=Helm
                        app.kubernetes.io/name=metrics-server
                        app.kubernetes.io/version=0.6.1
                        helm.sh/chart=metrics-server-3.8.2
Annotations:            deployment.kubernetes.io/revision: 1
                        meta.helm.sh/release-name: metrics-server
                        meta.helm.sh/release-namespace: kube-system
Selector:               app.kubernetes.io/instance=metrics-server,app.kubernetes.io/name=metrics-server
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app.kubernetes.io/instance=metrics-server
                    app.kubernetes.io/name=metrics-server
  Service Account:  metrics-server
  Containers:
   metrics-server:
    Image:      k8s.gcr.io/metrics-server/metrics-server:v0.6.1
    Port:       4443/TCP
    Host Port:  0/TCP
    Args:
      --secure-port=4443
      --cert-dir=/tmp
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --kubelet-use-node-status-port
      --metric-resolution=15s
    Liveness:     http-get https://:https/livez delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get https://:https/readyz delay=20s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /tmp from tmp (rw)
  Volumes:
   tmp:
    Type:               EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:          <unset>
  Priority Class Name:  system-cluster-critical
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   metrics-server-7d76b744cd (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m50s  deployment-controller  Scaled up replica set metrics-server-7d76b744cd to 1
```
查看 Pod log
```
kubectl logs -f -n kube-system metrics-server-7d76b744cd-fv9ns
```

![](https://imgur.com/1NaduDA.png)

## 連線失敗問題修正
加上 `--kubelet-insecure-tls` 啟動參數
```
kubectl patch -n kube-system deployment metrics-server --type=json -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

## 測試
查看節點資源消耗
```
[root@node ~]# kubectl top nodes
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node      312m         7%     2769Mi          36%
rockyw    234m         2%     3335Mi          21%
rockyw2   199m         2%     2727Mi          17%
```

## Reference
在找資料的時候發現以下兩篇針對 metric server 的介紹，值得拜讀。 
- [Kubernetes Monitoring 101 — Core pipeline & Services Pipeline](https://levelup.gitconnected.com/kubernetes-monitoring-101-core-pipeline-services-pipeline-a34cd4cc9627)
- [Kubernetes 自動彈性伸縮](https://www.xtplayer.cn/kubernetes/k8s-automatic-elastic-expansion/r)