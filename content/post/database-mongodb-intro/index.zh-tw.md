---
title: 在 Kubernetes 上佈署 MongoDB
tags:
  - Kubernetes
  - MongoDB
categories:
  - DataEngineering
  - Database
  - Kubernetes
date: 2022-04-20 19:59:00
slug: install-mongodb-on-kubernetes
---

紀錄一下 mongodb 的 kubernetes 佈署檔，分為 standalone 以及 replica set，會宣告 persistent volume 以儲存永久性資料 (如 database 資料、index 以及設定檔)。

<!--more-->

\*\*假設已建立以 nfs 為基底的 storage class。

## Standalone

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-mongo-pvc
  namespace: test-mongo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs

---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: mongodb
  namespace: test-mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo:latest
          command:
            ["mongod", "--bind_ip_all", "--noauth", "--dbpath", "/data/db"]
          ports:
            - containerPort: 27017
              name: mongodb
              protocol: TCP
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: mongodb-data
              mountPath: /data/db
      volumes:
        - name: mongodb-data
          persistentVolumeClaim:
            claimName: test-mongo-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: test-mongo
  labels:
    name: mongodb
spec:
  ports:
    - port: 27017
      name: mongo-port
      protocol: TCP
      targetPort: 27017
  selector:
    app: mongodb
  type: NodePort
```

## Replicaset

(待補充)
