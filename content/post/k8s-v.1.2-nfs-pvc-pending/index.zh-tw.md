---
title: 'k8s v1.20 nfs-client-provisioner 創建 pvc 時停在 Pending'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2021-08-29 21:47:00
slug: kubernets-nfs-client-provisioner-pending-in-creating-pvc
---
上次將 K8s 集群從 1.7 升級到 1.20 之後，在創建 pvc 時，發現狀態會一直停留在 Pending，詳細資訊如下：

<!--more-->

```bash
[root@master1 telegraf]# kubectl get pvc
NAME   STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
test   Pending                                      nfs            5s
[root@master1 telegraf]# kubectl describe pvc test
Name:          test
Namespace:     default
StorageClass:  nfs
Status:        Pending
Volume:
Labels:        <none>
Annotations:   volume.beta.kubernetes.io/storage-provisioner: cluster.local/nfs-sc-0-nfs-client-provisioner
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                From                         Message
  ----    ------                ----               ----                         -------
  Normal  ExternalProvisioning  10s (x5 over 50s)  persistentvolume-controller  waiting for a volume to be created, either                                                        by external provisioner "cluster.local/nfs-sc-0-nfs-client-provisioner" or manually created by system administrator
```

查了原因可能是因為 selfLink 為空，無法建立參考。
Kubernetes v1.20 開始，根據 [change log](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#other-cleanup-or-flake-2) 說明，默認刪除了 metadata.selfLink 字段，然而部分應用仍然依賴於此功能，例如 nfs-client-provisioner。如果仍然要繼續使用這些應用，需要重新啟用selfLink。

## 解決方式

修改 kube-apiserver 文件

```bash
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```

在 command 下的參數加上 `--feature-gates=RemoveSelfLink=false`

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.1.5.141
    - --allow-privileged=true
    - --au...
    .
    .
    .
    **- --feature-gates=RemoveSelfLink=false**
```

儲存編輯後，kube-apiserver 便會自動重啟。