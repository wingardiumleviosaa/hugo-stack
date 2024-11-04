---
title: Kubernetes 升級紀錄
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2021-08-22 21:29:00
slug: k8s-upgrade
---
紀錄在現有 kubernetes 1.17 集群升級到 1.20 的過程。

<!--more-->

## 環境說明
- 集群配置

|     主機名     |     系統                            |     IP Address    |
|----------------|-------------------------------------|-------------------|
|     vip        |     Server   Load Balancer (SLB)    |     10.1.5.140    |
|     master1    |     CentOS 7.8                      |     10.1.5.141    |
|     Worker1    |     CentOS 7.8                      |     10.1.5.142    |

- 當前運行版本

|     組件                            |     版本        |
|-------------------------------------|-----------------|
|     kubeadm                         |     v1.17.13    |
|     kubelet                         |     v1.17.13    |
|     kubectl                         |     v1.17.13    |
|     Container   Runtime - Docker    |     v20.10.7    |
|     etcd                            |     V3.4.3-0    |
|     kube-apiserver                  |     v1.17.17    |
|     kube-controller-manager         |     v1.17.17    |
|     kube-proxy                      |     v1.17.17    |
|     kube-scheduler                  |     v1.17.17    |
|     coredns                         |     v1.6.5      |
|     pause                           |     v3.1        |

查看版本的命令如下
```bash
kubeadm version
kubelet --version
kubectl version
kubectl get node master1 -o yaml
```

## Kubernetes Non-Active Branch History
目前使用的 1.17 版已在 2021-01-13 EOL，final patch release 在 1.17.17，請參考官方 [repo]( https://github.com/kubernetes/website/blob/main/content/en/releases/patch-releases.md#non-active-branch-history) 說明。

## 準備升級
### 版本升級的必要
- 對於 Kubernetes 集群的使用者：

更新的版本能有更新的功能、更加全面的安全補丁以及諸多的bugfix。
- 對於 Kubernetes 集群的運維者：

通過集群升級功能可以拉齊所管理的集群版本，減少集群版本的碎片化，從而減少管理成本和維護成本。

### 升級注意事項
- 升級僅支持一個小版本號。也就是說，只能從 1.7 升級到 1.8 的最新版本，或是從 1.17.13 升級到 1.17.17，而不能從 1.7 直接升級到 1.9。
- 一次更新一個節點，確保 Kubernetes 功能不會因為更新而中斷。
- 升級後所有容器都會被重啟，避免服務中斷，需確保應用程式使用進階的 Kubernetes API 建立，如 Deployment，或是利用副本機制。
- kubeadm upgrade 不會影響工作負載，只會涉及Kubernetes 內部的組件，但為保險起見，仍然可以先備份 etcd 的狀態。

### 備份 etcd
> Snapshot 又稱為快照，就像照相一樣，在某個時間點，將硬碟目前的整個狀態儲存起來，以作為將來還原的備份依據。
Snapshot 是幾乎所有的儲存服務設備都會提供的功能，就像是幫你硬碟上的資料照張像一樣，把這個目前的狀態記錄下來，以備將來還原之用。

`etcd` 的備份有兩種方式：

1. 使用 etcdctl snapshot save 進行備份
```bash
$ kubectl -n kube-system exec -it etcd-master1 -- sh -c "ETCDCTL_API=3 ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key etcdctl --endpoints=https://127.0.0.1:2379 snapshot save /var/lib/etcd/snapshot1.db"
```
使用 etcdctl snapshot status查看備份
```bash
$ kubectl -n kube-system exec -it etcd-master1 -- sh -c "ETCDCTL_API=3  ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key etcdctl --endpoints=https://127.0.0.1:2379 snapshot status -w table /var/lib/etcd/snapshot.db"
```
</br>  

![](https://imgur.com/0nBoLHn.png)

檢視備份檔
```bash
$ ls /var/lib/etcd/
member  snapshot.db
$ mkdir ~/backup
$ cp /var/lib/etcd/snapshot.db ~/backup/	
# 備份 etcd 核心資料檔案
$ cp -r /etc/kubernetes/pki/etcd $HOME/backup/
```

2. 使用 docker etcd image 連線進入 etcd 內部下 command

```bash
$ mkdir -vp /data/backup
$ docker run --rm                                    \
-v /data/backup:/backup                              \
-v /etc/kubernetes/pki/etcd:/etc/kubernetes/pki/etcd \
--env ETCDCTL_API=3                                  \
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.13-0 \
/bin/sh -c "etcdctl --endpoints=https://192.168.12.226:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt                  \
--key=/etc/kubernetes/pki/etcd/healthcheck-client.key     \
--cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt    \
snapshot save /backup/etcd-snapshot-1.19.13.db"
```

## 開始升級

### 1.17.13 ~ 1.18.20

#### 主節點

確認可升級版本與升級方案
```bash
$ yum list --showduplicates kubeadm --disableexcludes=kubernetes
```

通過以上命令查詢到 1.18 當前最新版是 1.18.20-0 版本。master 如有三個節點可先從 master3 節點開始升級。

![](https://imgur.com/yVk7FeI.png)

yum 升级 kubernetes 插件

```bash
$ yum install kubeadm-1.18.20-0 kubelet-1.18.20-0 kubectl-1.18.20-0 --disableexcludes=kubernetes
```
騰空節點檢查集群是否可升級
```bash
$ kubectl drain master1 --ignore-daemonsets
$ kubeadm upgrade plan
```
</br>

![](https://imgur.com/tthRTKH.png)

升級
```bash
$ kubeadm upgrade apply v1.18.20
```
</br>

![](https://imgur.com/OMmi0Jk.png)

重啟 kubelet 取消節點保護

```bash
$ systemctl daemon-reload
$ systemctl restart kubelet
$ kubectl uncordon master1
$ kubectl get nodes
```
</br>

![](https://imgur.com/5jbXQGI.png)

經測試後雖然測試環境的集群的 master 只有一個，但升級單一這個 master 後好像不會造成 Pod 的影響。

![](https://imgur.com/w0o2djB.png)

#### 工作節點

切換到 worker node，yum升级kubernetes插件
```bash
# 在 worker node 節點操作
$ ssh worker
$ yum install kubeadm-1.18.20-0 kubelet-1.18.20-0 kubectl-1.18.20-0 --disableexcludes=kubernetes
```
將節點標記為不可調度並逐出工作負載。

```bash
# master1
$ kubectl drain worker1 --ignore-daemonsets

# 可以看見與下類似的輸出：
node/ip-172-31-85-18 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-dj7d7, kube-system/weave-net-z65qx
node/ip-172-31-85-18 drained
```
升級
```bash
# worker1
$ kubeadm upgrade node
```
重啟kubelet 並取消對節點的保護
```bash
# worker1
$ systemctl daemon-reload
$ systemctl restart kubelet
# master1
$ kubectl uncordon worker1
# 通過將節點標記為可調度，讓節點重新上線，如果有做保護節點的步驟的話在做。
$ kubectl uncordon worker1
```

### 從 1.18.20 升級到 1.19.12
重複以上動作
### 從 1.19.12 升級到 1.20.10
重複以上動作


## Reference
- [https://cloud.tencent.com/developer/article/1848150](https://cloud.tencent.com/developer/article/1848150)
- [https://v1-18.docs.kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/](https://v1-18.docs.kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [https://xie.infoq.cn/article/6e47d96d4c4a4c6d47852adc8](https://xie.infoq.cn/article/6e47d96d4c4a4c6d47852adc8)
- [https://www.796t.com/article.php?id=260536](https://www.796t.com/article.php?id=260536) for etcd backup