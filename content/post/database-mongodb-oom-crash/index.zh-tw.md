---
title: Mongodb 可能是讓機器 OOM crash 的元兇 ?!
tags: ["MongoDB"]
categories:
  - DataEngineering
  - Database
date: 2022-03-14 19:53:00
slug: mongodb-oom-crash
---

## 問題

手上有三台各有 160GB 的機器，但卻會在跑一段時間後輪流死當。觀察記憶體消耗之後，發現兇手就是 MongoDB!!!

<!--more-->

![](https://imgur.com/HyeoJGH.png)

## 原因

MongoDB 使用記憶體映射存儲引擎 (Memory Mapped Storage Engine) WiredTiger，是 MongoDB 3.2 版之後的默認引擎，它會把磁盤 IO 操作轉換成記憶體操作，如果是讀操作，記憶體中的數據起到緩存的作用，如果是寫操作，記憶體還可以把隨機的寫操作轉換成順序的寫操作，總之可以大幅度提升性能。
截自 [Mongodb doucument](https://docs.mongodb.com/manual/core/wiredtiger/#memory-use)

```
With WiredTiger, MongoDB utilizes both the WiredTiger internal cache and the filesystem cache.
Starting in MongoDB 3.4, the default WiredTiger internal cache size is the larger of either:
- 50% of (RAM - 1 GB), or
- 256 MB.
```

因為伺服器記憶體大小為 160GB，一個 Mongodb 就可能會占掉近 80GB 的記憶體。 此時服務器上若還有跑其他 Mongodb 或應用程序的話，就會導致記憶體不足而退出。

## 解決方法

### 在 mongodb 啟動時設定

- run on baremetal

```
mongod --wiredTigerCacheSizeGB 10
```

- run on docker

```
docker run --name mymongo -d mongo --wiredTigerCacheSizeGB 10
```

- run on kubernetes

```yaml
cat <<EOF | kubectl apply -f -
kind: Deployment
apiVersion: apps/v1
metadata:
  name: mongodb
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
        - image: mongo
          name: mongodb
          ports:
          - containerPort: 27017
            name: mongodb
            protocol: TCP
          imagePullPolicy: IfNotPresent
          command: ["docker-entrypoint.sh"]
          args: ["mongod", "--wiredTigerCacheSizeGB", "10"]
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
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
EOF
```

### mongodb 運行時設定

如果 mongodb 已經啟動的話，可以使用下面方式動態設定

```sql
db.adminCommand({setParameter: 1, wiredTigerEngineRuntimeConfig: "cache_size=10G"})
```

![](https://imgur.com/YTtSbvo.png)

### 查看設定結果

```sql
db.serverStatus().wiredTiger.cache['maximum bytes configured']/1024/1024/1024
```

![](https://imgur.com/cmRSf8R.png)

## Reference

- https://docs.mongodb.com/manual/reference/program/mongod/
- https://stackoverflow.com/questions/64809287/how-to-set-wiredtigercachesize-in-mongodb-when-deployed-in-kubernetes/64859613#64859613
- https://blog.csdn.net/LuyaoYing001/article/details/75576820
