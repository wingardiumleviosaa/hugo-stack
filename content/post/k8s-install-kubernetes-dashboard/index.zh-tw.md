---
title: 安裝 Kubernetes Dashboard - 單集群可視化管理
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-03-13 14:46:00
slug: kubernets-dashboard-installation
---

Kubernetes Dashboard 是由官方維護的 Kubernetes 集群 WEB UI 管理工具，能查看 Kubernetes Cluster 上資源分佈與使用狀況，也可以創建或者修改 Kubernetes 資源，讓使用者透過 Web UI 介面取代指令的管理 Kubernetes。

<!--more-->

## 安裝
安裝非常簡單，只要透過下面 command 即可部屬。
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml
```
確認安裝結果
```
[root@ula ~]# kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-799d786dbf-zhvlc   1/1     Running   0          24s
pod/kubernetes-dashboard-fb8648fd9-wd4t5         1/1     Running   0          24s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.111.121.48    <none>        8000/TCP   24s
service/kubernetes-dashboard        ClusterIP   10.107.248.188   <none>        443/TCP    25s

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           24s
deployment.apps/kubernetes-dashboard        1/1     1            1           25s

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-799d786dbf   1         1         1       24s
replicaset.apps/kubernetes-dashboard-fb8648fd9         1         1         1       24s
```

## 訪問
將 kubernetes-dashboard 服務暴露 NodePort
```
kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard
```
將原本 `type: ClusterIP` 改成 `type: NodePort`。完成後就可以使用 `https://NodeIP:nodePort` 地址訪問 dashboard。
```
[root@ula ~]# kubectl get svc -n kubernetes-dashboard kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.107.248.188   <none>        443:31302/TCP   44m
```
> 由於 Dashboard 默認使用 https，其證書不受瀏覽器信任，所以訪問時加上 https 強制跳轉就可以了。

![](https://imgur.com/Mw9Xl0U.png)

## 登入
登錄 Dashboard 支持 Kubeconfig 和 Token 兩種認證方式，Kubeconfig 中也依賴 token 字段，所以生成 token 這一步是必不可少的。下面紀錄使用 token 的方式登錄。

### 建立 service account & role binding
準備 yaml 檔 sc-ula.yaml
```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: ula
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin # k8s 預設建立的角色
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: ula
  namespace: kube-system

  ---
  apiVersion: v1
  kind: ServiceAccount
  metadata:
  name: ula
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```
使用 kubectl apply 建立
```
kubectl apply -f sc-ula.yaml
```

### 取得 Server Account Token
查看 service account secret
```yaml
kubectl get sa ula -n kube-system -o=yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2022-03-14T03:12:41Z"
  name: ula
  namespace: kube-system
  resourceVersion: "332586"
  uid: f048e242-5947-4945-8310-432628e635b7
secrets:
- name: ula-token-qwddr
```
取得 token 字段，並使用 base64 decode
```
kubectl get secret ula-token-qwddr -o jsonpath={.data.token} -n kube-system | base64 -d
eyJhbGciOiJSUzI1NiIsImtpZCI6IjVadWpvSGEtR2tMR2ZFMGVHaGloWjJNOFdRcn...............
```
然後在 dashboard 登錄頁面上直接使用上面得到的 token 字符串即可登錄，這樣就可以擁有管理員權限操作整個 kubernetes 集群的對象，也可以新建一個指定操作權限的用戶。

![](https://imgur.com/1FymCmE.png)

## Reference
- https://github.com/kubernetes/dashboard
- https://kubernetes.io/zh/docs/tasks/access-application-cluster/web-ui-dashboard/