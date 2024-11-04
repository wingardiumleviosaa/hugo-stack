---
title: '[Kubernetes] 停止調度 / 刪除節點'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-05-19 11:36:00
slug: kubernetes-delete-worker-node
---

cordon、drain 和 delete 三個命令都會使 kubernetes node 停止被調度，本篇記錄如何優雅的刪除節點。

<!--more-->

## Drain the Node
使用 kubectl drain 將 Node 狀態變更為維護模式，該 Node 上面的 Pod 就會轉移到其他 Node 上。
```bash
kubectl drain worker3 --ignore-daemonsets --delete-local-data
```
kubectl drain 操作會將指定節點上的 Pod 刪除，並在可調度節點上面起一個對應的 Pod。當舊 Pod沒有被正常刪除的情況下，新 Pod 不會起來。例如舊 Pod 一直處於Terminating 狀態。所以可以強制刪除該 Pod。
```
kubectl get pod all -A -o wide | grep worker3

kubectl delete pods -n my-kafka-project my-cluster-zookeeper-2  --force
```
## Delete the Node
```
kubectl delete nodes worker3
```
delete 是一種暴力刪除 node 的方式，會強制關閉容器進程以驅逐 pod。
基于 node 的[自註冊](https://kubernetes.io/docs/concepts/architecture/nodes/#self-registration-of-nodes)功能，恢復調度則重啟 kubelet 服務即可。
```
systemctl restart kubelet
```

## Drain v.s Cordon

### cordon 停止調度（不可調度，從 K8S 集群隔離）
只會將 node 標識為SchedulingDisabled 不可調度狀態。新創建的資源，不會被調度到該節點。而舊有的 pod 不會受到影響，仍正常對外提供服務。
```bash
# 禁止調度
kubectl cordon <nodeName>
# 恢復調度
kubectl uncordon <nodeName>
```
### drain 驅逐節點（先不可調度，然後排乾 Pod）
會驅逐 Node 上的 pod 資源到其他節點重新創建。接著，將節點調為 SchedulingDisabled 不可調度狀態。
```bash
# 禁止調度
kubectl drain <nodeName> --force --ignore-daemonsets --delete-local-data
# 恢復調度
kubectl uncordon <nodeName>
```
- `--force` 當一些 pod 不是經ReplicationController, ReplicaSet, Job, DaemonSet 或者 StatefulSet 管理的時候就需要用 --force 來強制執行(例如 kube-proxy)
- `--ignore-daemonsets` 無視DaemonSet 管理下的 Pod。因為deamonset 會忽略 unschedulable 標籤，因此 deamonset 控制器控制的 pod 被刪除後可能馬上又在此節點上啟動起來，這樣就會成為死循環，因此這裡忽略daemonset。
- `--delete-local-data` 如果有 mount local volumn 的 pod，會強制殺掉該 pod。
drain 驅逐流程：先在 Node 節點優雅關閉並刪除 pod，然後再在其他 Node 節點創建該 pod。所以為了確保 drain 驅逐 pod 過程中不中斷服務，必須保證要驅逐的 pod 副本數大於 1，並且採用了 anti-affinity 將這些 pod 調度到不同的 Node 節點上。


## Reference
- https://www.cnblogs.com/kevingrace/p/14412254.html
