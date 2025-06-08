---
title: '[MongoDB] 無法使用 replicaSet 連線問題'
categories:
  - DataEngineering
  - Database
tags: ["MongoDB"]
date: 2020-12-20 20:07:00
slug: "mongodb-replicaSet-connection-failed"
---
## 前言
用 docker 跑三個 mongoDB 的 container，做成 replica set cluster。但卻遇到從外面的 shell (跑在本機或其他 server) 加上 replica set 的名字時無法連線的問題 (`mongo --host myReplica/mongo1` 失敗)。而不加 replica set 卻都能成功連線 (`mongo --host mongo1:30001` 成功)。
<!--more-->

## 原因
外部連線的 hostname 跟 port 與 mongodb 內部的 replica set configuration 不同

## 解決辦法
參考 [這篇 issue](https://github.com/bitnami/bitnami-docker-mongodb/issues/84) 後發現有兩種做法，一種是在建立 container 的時候為每個角色指定環境變數 (如 `MONGODB_REPLICA_SET_MODE=primary`)。但這個做法總覺得哪裡怪怪的，不太確定在環境變數指定 primary 或 secondary 後，當 primary 掛掉時環境變數固定會不會造成什麼影響。

所以我就用第二種方法，把 container 內的 mongod 的 port 與外面 export 的 port 設成一樣的。然後連線前在 /etc/hosts 中設定對應的 hostname。

```s
docker run \
-p 30001:30001 \
--name mongo1 \
--network mongo-cluster \
mongo mongod --replSet myReplica --port 30001
```

這樣子 `rs.conf().members` 的 ip & port 才和外面會一樣。
```s
[root@test ~]# mongo --host myReplica/mongo1:30001,mongo2:30002,mongo3:30003
myReplica:PRIMARY> rs.conf().members
[
        {
                "_id" : 0,
                "host" : "mongo1:30001",
                "arbiterOnly" : false,
                "buildIndexes" : true,
                "hidden" : false,
                "priority" : 1,
                "tags" : {

                },
                "slaveDelay" : NumberLong(0),
                "votes" : 1
        },
        {
                "_id" : 1,
                "host" : "mongo2:30002",
                "arbiterOnly" : false,
                "buildIndexes" : true,
                "hidden" : false,
                "priority" : 1,
                "tags" : {

                },
                "slaveDelay" : NumberLong(0),
                "votes" : 1
        },
        {
                "_id" : 2,
                "host" : "mongo3:30003",
                "arbiterOnly" : false,
                "buildIndexes" : true,
                "hidden" : false,
                "priority" : 1,
                "tags" : {

                },
                "slaveDelay" : NumberLong(0),
                "votes" : 1
        }
]
```

## Reference
- 文中的 github issue 連結 https://github.com/bitnami/bitnami-docker-mongodb/issues/84
- 這篇也解釋得很清楚，可以參考參考 https://serverfault.com/questions/895355/docker-container-unable-to-connect-to-mongodb-replica-set