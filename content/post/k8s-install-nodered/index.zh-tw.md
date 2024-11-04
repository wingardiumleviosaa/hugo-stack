---
title: '[Node-RED] Deploy on Kubernetes'
tags:
  - Node-RED
  - Kubernetes
categories:
  - Kubernetes
date: 2021-10-26 09:44:00
slug: install-nodered-on-kubernetes
---
原先使用 k8s-at-home 的 [helm chart](https://github.com/k8s-at-home/charts/tree/master/charts/stable/node-red) 部屬，但完成後發現 node 安裝後會 deploy 異常，懷疑是 persistence 設定問題，但又不想花時間深究，所以就直接自己寫 yaml 部屬比較快。

<!--more-->

1. 準備 node-red.yaml 檔
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nodered-pvc
  namespace: node-red
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-red
  namespace: node-red
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-red
  template:
    metadata:
      labels:
        app: node-red
    spec:
      containers:
        - name: node-red
          image: nodered/node-red:2.1.2
          ports:
          - containerPort: 1880
            name: node-red
            protocol: TCP
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: nodered-data
              mountPath: /data
      volumes:
        - name: nodered-data
          persistentVolumeClaim:
            claimName: nodered-pvc

---

apiVersion: v1
kind: Service
metadata:
  name: node-red
  namespace: node-red
spec:
  type: NodePort
  selector:
    app: node-red
  ports:
  - protocol: TCP
    port: 1880
    nodePort: 31880

```

2. 部屬
```bash
kubectl create ns node-red
kubectl -n node-red apply -f node-red.yaml 
```

3. 完成
```bash
kubectl get all -n node-red
```

![](https://imgur.com/VCHgLRn.png)

4. 訪問
service 使用 NodePort，則直接使用 control plane 的 IP 以及 NodePort 指定的 port 31880 訪問 Node-RED 即可。

![](https://imgur.com/fcezxst.png)