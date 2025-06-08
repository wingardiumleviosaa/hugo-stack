---
title: "[MongoDB] Introduction and Installation on Bare Metal"
categories:
  - DataEngineering
  - Database
tags: ["MongoDB"]
date: 2020-12-15 21:55:00
slug: "mongodb-introduction-and-installation"
---

文件導向式(document-oriented)資料庫，儲存文件（document）或物件（object），沒有 row 的概念，取而代之的是 document。

<!--more-->

## Features

- schema-less：擁有彈性的 schema
- 易於橫向擴展，document 的數據模型能很容易在多台伺服器之間進行數據分割。
- 性能：MongoDB 能預分配，以利用額外的空間換取穩定，同時盡可能把多的內存用作 cache，試圖為每次查詢自動選擇正確的索引。查詢(使用索引)、插入(自動分片)等操作的速度很快。
- Map/Reduce 的聚合（aggregation）功能：更多可能性，且提供方便的資料分組、處理與二次加工等操作
- 副本（Replication）& 容錯移轉（failover）：提供資料的高可用性（HA, High Availability）

## 缺點

不支援事務操作 : 所以通常不適合應用在銀行或會計這種系統上，因為不包證一致性。
占用比較多空間 : 主要是有兩個原因，首先是它會預分配空間，為了提高效能，而第二個原因是欄位所占用的空間。

## 儲存架構

### document

Document 是 mongodb 的核心，它就是 Key-Value 的對應組合。資料的儲存架構是以類似 JSON 的資料結構 **BSON**(Binary JSON) 儲存，每筆資料的 key 和 value 都是區分大小寫。

```json
{
  "_id": "948794777",
  "name": "Robby",
  "age": 30,
  "email": "Robby",
  "skill": ["javascript", "java"]
}
```

- \_id：雖然稱為 NoSQL，但系統會自動幫你產生
- key 區分大小寫，且不能相同

### collection

Collection 是一組 Document，如果把它用來與關聯式資料庫比較，他就是 Table 裡面存放了很多 Row (document)。Collection 是動態的，一個 collection 裡的 document 可以是各種類型。如下；

```json
{ id : 1, name : "mark" }
{ age : 100 }
```

## 在 Ubuntu 18.04 上安裝 MongoDB 4.4 Community Edition

{{< notice info >}}
MongoDB 4.4 Community Edition 僅支援 64-bit 的 20.04、18.04、16.04 Ubuntu 版本。
官方的套件是由 MongoDB Inc. 維護的 mongodb-org，並且會在 repo 中保持最新版本。而 Ubuntu 提供的套件是 mongodb 非為官方套件，在安裝官方套件前須前解除安裝。
{{< /notice >}}

### 導入 apt 使用的公鑰

```
$ wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```

如果成功會返回 `OK`；失敗的話請先安裝 `gnupg`，`sudo apt-get install gnupg -y`。

### 新增 MongoDB 的 list file

```
$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```

成功後會在 /etc/apt/sources.list.d 下建立 mongodb-org-4.4.list 清單。

### 更新 apt 並安裝 mongo

```
$ sudo apt-get update
$ sudo apt-get install -y mongodb-org
```

安裝過程會自動創建使用者 `mongodb` 以及數據目錄 `/var/lib/mongodb` 以及日誌目錄 `/var/log/mongodb`。如果要更改使用 mongod 進程的使用者的話，則這兩個目錄的權限也要修改。

### 固定版本

為了防止在 apt update 時將最新版本下載更新，所以可以選擇將 package 固定在當前安裝的版本上。

```
$ echo "mongodb-org hold" | sudo dpkg --set-selections && echo "mongodb-org-server hold" | sudo dpkg --set-selections && echo "mongodb-org-shell hold" | sudo dpkg --set-selections && echo "mongodb-org-mongos hold" | sudo dpkg --set-selections && echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

### 設定 ulimit

大多數類 Unix 操作系統都限制 process 可使用的系統資源。這些限制可能會對 MongoDB 的運作造成停擺。在 4.4 版如果 ulimt 的 open file 數值小於 64000 會直接報 startup error 的錯誤。
先使用 ps 查看 mongo 的使用者名稱，

```
$ ps aux | grep mongod
```

![](https://imgur.com/SAZKb1H.png)

修改 `limit.conf` 檔案

```
$ sudo vi /etc/security/limits.conf
```

加入以下條件：

```
mongodb soft fsize unlimited
mongodb hard fsize unlimited
mongodb soft cpu unlimited
mongodb hard cpu unlimited
mongodb soft as unlimited
mongodb hard as unlimited
mongodb soft memlock unlimited
mongodb hard memlock unlimited
mongodb soft nofile 64000
mongodb hard nofile 64000
mongodb soft nproc 64000
mongodb hard nproc 64000
```

之後重登入使用者，並可以使更改生效。

### mongod.conf

設置檔在啟動時生效。如果在運行 MongoDB 實例時更改配置文件，則必須重新啟動服務以使更改生效。

```
# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1
  # 綁定監聽的 ip，默認 127.0.0.1，只能通過本地連接。若不限制 IP，務必確保認證安全，多個 IP 用逗號分隔。

# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo
```

### 啟動 mongodb service

```
$ sudo systemctl daemon-reload
$ sudo systemctl start mongod
```

確認啟用狀態

```
$ sudo systemctl status mongod
```

確保 MongoDB 會在系統重啟後自動啟動

```
$ sudo systemctl enable mongod
```

### 進入 mongoDB

```
$ mongo
```

如果要有指定特定 port 進入 shell

```
$ mongo --port 3000
```

如果要連到 remote mongoDB

```
# 使用 connection string進入
$ mongo "mongodb://10.1.5.10:27017"
# 或是使用參數
$ mongo --host 10.1.5.10:27017
# 或是使用參數
$ mongo --host 10.1.5.10 --port 27017
```

使用驗證連到 mongo shell

```
# 使用 connection string 登入
$ mongo "mongodb://alice@10.1.5.10:27017/?authSource=admin"
# 或是使用參數登入
mongo --username alice --password --authenticationDatabase admin --host 10.1.5.10 --port 27017
```

不需要在 command line 裡把密碼打出來，shell 會自動跳出密碼輸入提示。

## Reference

- https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/
- https://ithelp.ithome.com.tw/articles/10184679
