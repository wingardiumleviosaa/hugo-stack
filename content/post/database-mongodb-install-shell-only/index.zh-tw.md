---
title: "[MongoDB] Install MongoDB Shell"
categories:
  - DataEngineering
  - Database
tags: ["MongoDB"]
date: 2020-12-20 20:03:00
slug: "mongodb-install-shell"
---

## Install Mongo Shell Only on Linux

mongo shell 已經包含在 MongoDB 服務器中。如果已經安裝了 DB，則 mongo shell 將安裝在與服務器 binary 文件相同的位置。 但如果只想從 MongoDB 服務器外單獨下載 mongo shell，可以按照以下步驟將 shell 作為獨立程序安裝：

<!--more-->

## 下載

進入 [MongoDB Coummunity Download Center](https://www.mongodb.com/try/download/community)，依照作業系統版本選擇 mongo shell，並把連結**複製**起來。

![](https://imgur.com/gijHvpl.png)

```
wget https://fastdl.mongodb.org/linux/mongodb-shell-linux-x86_64-rhel70-4.4.2.tgz
tar tzxvf mongodb-shell-linux-x86_64-rhel70-4.4.2.tgz
```

## 複製 mongo 執行檔到執行目錄

```
cd mongodb-shell-linux-x86_64-rhel70-4.4.2/bin
cp mongo /usr/local/bin
```

執行 `mongo` 指令測試是否成功

```
mongo
MongoDB shell version v4.4.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed: SocketException: Error connecting to 127.0.0.1:27017 :: caused by :: Connection refused :
connect@src/mongo/shell/mongo.js:374:17
@(connect):2:6
exception: connect failed
exiting with code 1
```

就成功了！（p.s. 返回錯誤是因為本機 127.0.0.1:27017 沒裝 MongoDB）

## Reference

- https://docs.mongodb.com/manual/mongo/
