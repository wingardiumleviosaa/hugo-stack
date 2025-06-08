---
title: "[RabbitMQ] 在 K8s cluster 上安裝叢集"
categories: ["DataEngineering", "Kubernetes"]
tags: ["RabbitMQ"]
date: 2020-12-30 17:31:00
slug: rabbitmq-on-k8s
---

## 先決條件

- kubernetes 1.17 版以上
- RabbitMQ image 3.8.8+
<!--more-->

## 安裝 operator

```sh
kubectl apply -f "https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml"
```

確認 CRD 部屬完成

```
kubectl get customresourcedefinitions.apiextensions.k8s.io | grep rabbitmq
rabbitmqclusters.rabbitmq.com                         2020-12-29T06:22:27Z
```

## 安裝 RabbitMQ Cluster

RabbitMQ server 透過 `RabbitmqCluster` 資源來建立，整個集群資源(如 pod, svc, statefulSet)都會建在同一個指定 namespace 下。

### 準備 yaml 檔

準備一個 yaml 檔來定義 `RabbitmqCluster` 資源：

```
mkdir rbmq && cd rbmq && touch rbmq.yaml
vim rbmq.yaml
```

定義了 cluster name 為 `rbmq`、指定放在 `rabbitmq` namespace 下，複本數指定為 `3`、service 類型為 `LoadBalancer` (前提是 k8s 集群有建置 LB 套件)、指定 storageClass 以及空間。

```yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: rbmq
  namespace: rabbitmq
spec:
  replicas: 3
  service:
    type: LoadBalancer
  persistence:
    storageClassName: standard
    storage: 20Gi
```

<br>

{{< notice warning >}}
persistence 可以不寫，但 k8s 集群必須事先設置 default 的 StorageClass，若無填寫則會使用 default storageClass。否則集群將不會被 scheduled 成功，因為 rabbitMQ 需要有 persistent volume。
{{< /notice >}}

p.s. 使用 `kubectl get sc` 取得目前 k8s 集群的 StorageClass 資源

![](https://imgur.com/2DecQC8.png)

### 開始建立

應用上一步驟定義好的 `RabbitmqCluster` 資源，如果不使用與 operator 相同的 namespace 的話，需要先自己創一個。

```sh
kubectl create ns rabbitmq
kubectl apply -f rbmq.yaml
```

驗證資源是否都有建立成功

```sh
kubectl get all -n rabbitmq
```

![](https://imgur.com/ejZ1MXz.png)

## 加入 TLS

### 建立 Secret

需先準備好一組含有 key 跟 cert 的 pem 格式的金鑰，如果沒有就按照下面指令創建。

```
openssl req -x509 -newkey rsa:2048 -sha256 -nodes -keyout key.pem -out cert.pem -days 3650
```

使用現有的金鑰做成 secret

```sh
kubectl create secret -n rabbitmq tls rbmq-tls-secret --cert=/root/cert.pem --key=/root/key.pem
```

### 修改配置

```sh
kubectl edit rabbitmqcluster rbmq
```

在 spec.tls.secretName 中加上剛剛的 secret。

```yaml
spec:
  tls:
    secretName: rbmq-tls-secret
```

保存後離開就可以看到集群上已經開啟了 tls。

## 進入 Rabbit Management

開啟瀏覽器，輸入 LoadBalancer 配置的 `ip:15672` 進入 Rabbit Management。

### 取得使用者帳密

admin user 的帳密儲存在 secret 中，資源名字為 cluster name 加上 `-default-user`，帳密皆以 base44 編碼儲存。使用以下命令取得帳號以及密碼：

```
kubectl -n rabbitmq get secret rbmq-default-user -o jsonpath="{.data.username}" | base64 --decode
kubectl -n rabbitmq get secret rbmq-default-user -o jsonpath="{.data.password}" | base64 --decode
```

## Reference

- https://www.rabbitmq.com/kubernetes/operator
