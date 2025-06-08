---
title: 安裝監聽於所有 namespace 的 strimzi kafka operator
date: 2022-03-21 22:33:25
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
slug: "install-strimzi-kafka-operator-to-watch-all-namespace"
---

以下紀錄如何隔離 kafka operator 與建立的 kafka cluster 的命名空間，預計是可以在一座 kubernetes cluster 上只需安裝一個 kafka operator 來建立多個 kafka cluster。

<!--more-->

## 安裝 operator

下載 CRD 資源

```bash
wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.28.0/strimzi-0.28.0.tar.gz -P /tmp && tar zxvf /tmp/strimzi-0.28.0.tar.gz -C /tmp
```

修改 `strimzi-0.28.0/install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml` 檔案中的 `STRIMZI_NAMESPACE` 的值，改為監聽所有 `"*"` namespace。

```yaml
# vim /tmp/strimzi-0.28.0/install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml
apiVersion: apps/v1
kind: Deployment
spec:
  # ...
  template:
    spec:
      # ...
      serviceAccountName: strimzi-cluster-operator
      containers:
        - name: strimzi-cluster-operator
          image: strimzi/operator:0.16.2
          imagePullPolicy: IfNotPresent
          env:
            - name: STRIMZI_NAMESPACE
              value: "*"
          # ...
```

以上可以用下面一行 command 解決

```bash
sed -z 's/              valueFrom:\n                fieldRef:\n                  fieldPath: metadata.namespace/              value: \"*\"/' -i /tmp/strimzi-0.28.0/install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml
```

建立 ClusterRoleBindings，使可授權 Cluster Operator 存取 cluster-wide 的所有 namespaces

```bash
kubectl create clusterrolebinding strimzi-cluster-operator-namespaced --clusterrole=strimzi-cluster-operator-namespaced --serviceaccount system-kafka-operator:strimzi-cluster-operator
kubectl create clusterrolebinding strimzi-cluster-operator-entity-operator-delegation --clusterrole=strimzi-entity-operator --serviceaccount system-kafka-operator:strimzi-cluster-operator
kubectl create clusterrolebinding strimzi-cluster-operator-topic-operator-delegation --clusterrole=strimzi-topic-operator --serviceaccount system-kafka-operator:strimzi-cluster-operator
```

正式佈署 operator

```
kubectl apply -f /tmp/strimzi-0.28.0/install/cluster-operator -n system-kafka-operator
```

查看資源

```
[root@node ~]# kubectl get all -n system-kafka-operator
NAME                                            READY   STATUS    RESTARTS   AGE
pod/strimzi-cluster-operator-7548bd689f-xlr9j   1/1     Running   0          29m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/strimzi-cluster-operator   1/1     1            1           29m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/strimzi-cluster-operator-7548bd689f   1         1         1       29m
```

## 安裝 kafka cluster

準備 kafka.yaml

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.1.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
      - name: external
        port: 9094
        type: loadbalancer
        tls: false
    config:
      default.replication.factor: 3
      num.partitions: 1
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      log.message.format.version: "3.1"
    storage:
      class: nfs
      type: jbod
      volumes:
        - id: 0
          type: persistent-claim
          size: 10Gi
          deleteClaim: false
  zookeeper:
    replicas: 3
    storage:
      class: nfs
```

佈署 cluster

```
kubectl apply -f kafka.yaml
```

查看資源

```
[root@node terraform]# kubectl get all -n kafka
NAME                         READY   STATUS    RESTARTS   AGE
pod/my-cluster-kafka-0       1/1     Running   0          25s
pod/my-cluster-kafka-1       1/1     Running   0          25s
pod/my-cluster-kafka-2       1/1     Running   0          25s
pod/my-cluster-zookeeper-0   1/1     Running   0          52s
pod/my-cluster-zookeeper-1   1/1     Running   0          52s
pod/my-cluster-zookeeper-2   1/1     Running   0          52s

NAME                                          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                               AGE
service/my-cluster-kafka-0                    LoadBalancer   10.109.143.19    10.1.5.151    9094:32608/TCP                        27s
service/my-cluster-kafka-1                    LoadBalancer   10.96.7.88       10.1.5.154    9094:32057/TCP                        27s
service/my-cluster-kafka-2                    LoadBalancer   10.99.44.55      10.1.5.152    9094:31265/TCP                        27s
service/my-cluster-kafka-bootstrap            ClusterIP      10.105.207.155   <none>        9091/TCP,9092/TCP,9093/TCP            27s
service/my-cluster-kafka-brokers              ClusterIP      None             <none>        9090/TCP,9091/TCP,9092/TCP,9093/TCP   27s
service/my-cluster-kafka-external-bootstrap   LoadBalancer   10.96.136.137    10.1.5.153    9094:30262/TCP                        27s
service/my-cluster-zookeeper-client           ClusterIP      10.107.192.225   <none>        2181/TCP                              53s
service/my-cluster-zookeeper-nodes            ClusterIP      None             <none>        2181/TCP,2888/TCP,3888/TCP            53s

NAME                                    READY   AGE
statefulset.apps/my-cluster-kafka       3/3     25s
statefulset.apps/my-cluster-zookeeper   3/3     52s
```

## Reference

- https://strimzi.io/docs/0.16.2/full.html#deploying-cluster-operator-to-watch-whole-cluster-deploying-co
