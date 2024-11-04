---
title: '[Kubernetes] Rook Health Warn with Clock Skew'
tags:
  - Kubernetes
  - Ceph
  - Rook
categories:
  - Kubernetes
date: 2024-05-01T22:20:00+08:00
slug: k8s-rook-ceph-clock-skew-detected
---

### 問題
rook-ceph 集群顯示 HEALTH WARN，其原因為 clock skew detected on mon.c, mon.d。
<!--more-->

```bash
kubectl -n rook-ceph describe cephcluster
```

```yaml
Status:
  Ceph:
    Capacity:
      Bytes Available:  1174133465088
      Bytes Total:      1181116006400
      Bytes Used:       6982541312
      Last Updated:     2024-04-16T01:21:15Z
    Details:
      MON_CLOCK_SKEW:
        Message:      clock skew detected on mon.c, mon.d
        Severity:     HEALTH_WARN
    Fsid:             caec8fab-28a0-464d-9079-463ddbb7c4e3
    Health:           HEALTH_WARN
    Last Changed:     2024-04-15T11:31:39Z
    Last Checked:     2024-04-16T01:21:15Z
```

### 解決

原本使用 ntpdate 強制每台都與指定的 ntp server 更新，但還有另一個方法是修改 mon clock drift allowed 預設參數

```yaml
kubectl edit configmap rook-config-override -n rook-ceph -o yaml
apiVersion: v1
data:
  config: |
    [global]
    mon clock drift allowed = 1
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: rook-ceph-cluster
    meta.helm.sh/release-namespace: rook-ceph
  creationTimestamp: "2024-04-12T05:38:23Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  name: rook-config-override
  namespace: rook-ceph
  ownerReferences:
  - apiVersion: ceph.rook.io/v1
    blockOwnerDeletion: true
    controller: true
    kind: CephCluster
    name: rook-ceph
    uid: a23004c1-fe54-4905-b9bf-af21a7b2a8ea
  resourceVersion: "994524"
  uid: 57ad721c-390a-4999-aa00-9f64e2497310
root@node1:~# kubectl get configmap rook-config-override -n rook-ceph -o yaml
apiVersion: v1
data:
  config: |
    [global]
    mon clock drift allowed = 1
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: rook-ceph-cluster
    meta.helm.sh/release-namespace: rook-ceph
  creationTimestamp: "2024-04-12T05:38:23Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  name: rook-config-override
  namespace: rook-ceph
  ownerReferences:
  - apiVersion: ceph.rook.io/v1
    blockOwnerDeletion: true
    controller: true
    kind: CephCluster
    name: rook-ceph
    uid: a23004c1-fe54-4905-b9bf-af21a7b2a8ea
  resourceVersion: "994524"
  uid: 57ad721c-390a-4999-aa00-9f64e2497310
```

新增了以下內容：

```yaml
config: |
    [global]
    mon clock drift allowed = 1
```

此處的時間可隨自身的時間差設置，在0.5到1s之間，不建議設定過大的值

接著刪除 mon Pod 使其載入新的設定文件

```bash
kubectl -n rook-ceph delete pod $(kubectl -n rook-ceph get pods -o custom-columns=NAME:.metadata.name --no-headers| grep mon)
pod "rook-ceph-mon-a-559d8b5866-dcmcz" deleted
pod "rook-ceph-mon-c-bfdbb5598-96kv8" deleted
pod "rook-ceph-mon-d-c9ff49c58-ml65k" deleted
```

查看狀態已恢復健康值

```bash
kubectl -n rook-ceph get cephcluster
NAME        DATADIRHOSTPATH   MONCOUNT   AGE     PHASE   MESSAGE                        HEALTH      EXTERNAL   FSID
rook-ceph   /var/lib/rook     3          3d19h   Ready   Cluster created successfully   HEALTH_OK              caec8fab-28a0-464d-9079-463ddbb7c4e3
```

### Reference

- https://blog.csdn.net/m0_59615922/article/details/131459393