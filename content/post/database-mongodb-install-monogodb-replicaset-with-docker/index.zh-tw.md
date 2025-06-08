---
title: "[MongoDB] Install MongoDB Replica Set with Docker"
categories:
  - DataEngineering
  - Database
tags: ["MongoDB"]
date: 2020-12-15 22:09:00
slug: "mongodb-install-db-replicaset-with-docker"
---

## Replica Set Installation

### Pre-requisties

確認是否安裝 Docker，並確認 docker daemon 是否有跑起來

<!--more-->

```
docker -v
docker images
```

下載最新版的官方 mongo image

```
docker pull mongo
```

### 架構

![](https://imgur.com/Q9CMsDT.png)

### 開始安裝

#### 設置網路

```
[root@test ~]# docker network ls

NETWORK ID     NAME            DRIVER    SCOPE
cd9c58204dfa   bridge          bridge    local
3954d16a34fc   host            host      local
146f0c7f58e7   none            null      local
```

#### 新增網路

```
docker network create mongo-cluster
```

#### 跑 container

```
docker run -d --name mongo1 --network mongo-cluster -p 30001:30001 -v mongo1:/data/db mongo:latest mongod --replSet myReplica --port 30001 --dbpath /data/db

docker run -d --name mongo2 --network mongo-cluster -p 30002:30002 -v mongo2:/data/db mongo:latest mongod --replSet myReplica --port 30002 --dbpath /data/db

docker run -d --name mongo3 --network mongo-cluster -p 30003:30003 -v mongo3:/data/db mongo:latest mongod --replSet myReplica --port 30003 --dbpath /data/db
```

#### 進入 container 設置 replication

```
docker exec -it mongo1 mongo --port 30001
```

<br>

```
> rs.initiate({"_id":"myReplica","members":[{"_id":0,"host":"mongo1:30001"},{"_id":1,"host":"mongo2:30002"},{"_id":2,"host":"mongo3:30003"}]})
```

第一個參數的 `_id` 必須與跑 mongod 的 --replSet 參數相同，第二個參數 `members` 需列出所有欲加入 replica set 的 node。而因為所有 container 都加在 mongo-cluster 的 docker network，所以可以直接使用 container name 辨別。

```
{
"ok" : 1,
"$clusterTime" : {
"clusterTime" : Timestamp(1608026586, 1),
"signature" : {
"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
"keyId" : NumberLong(0)
}
},
"operationTime" : Timestamp(1608026586, 1)
}
myReplica:SECONDARY>
myReplica:PRIMARY>
```

成功初始化群集後，應該可以看到回傳的訊息顯示 `ok = 1`。並且 shell 最前面會先變成 `myReplica:SECONDARY>`，再按一下 enter，就能成功變成 `myReplica:PRIMARY>`。先是 SECONDARY 的原因是因為還沒選出 PRIMARY。

#### 開始寫資料

```
> use test
> db.col1.insert({hello:'world'})
WriteResult({ "nInserted" : 1 })
> db.col1.find()
{ "_id" : ObjectId("87362127767493de37ff95ed"), "hello" : "world" }
```

再試試看連到第二台 MongoDB 找資料是不是已經備份成功了。在使用 Secondary 資料庫搜尋前，需要先下 `db2.setSlaveOk()` 讓 shell 知道我們有意要用非 primary 資料庫查詢。

```
> db2 = (new Mongo('mongo2:30002')).getDB('test')
test
> db2.setSlaveOk()
> db2.col1.find()
{ "_id" : ObjectId(87362127767493de37ff95ed"), "hello" : "world"  }
```

## Reference

- https://www.sohamkamani.com/blog/2016/06/30/docker-mongo-replica-set/
