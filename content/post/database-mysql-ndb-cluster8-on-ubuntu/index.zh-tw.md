---
title: "[MySQL] 在 Ubuntu18.04 建立 MySQL NDB Cluster 8.0"
categories:
  - DataEngineering
  - Database
tags: ["MySQL"]
date: 2020-08-11 11:51:00
slug: "mysql-ndb-cluster"
---

## MySQL NDB Cluster

MySQL cluster 將標準的 MySQL server 與內存中叢集式儲存引擎 NDB 集成起來，允許在多個無共享的系統中部署「內存中」資料庫的叢集。

<!--more-->

叢集通常會有多台機器，每台運行不同程序，包括 MySQL Server、NDB 叢集的數據節點、管理伺服器，以及可能專門的數據訪問程式，如下圖：

![](https://imgur.com/tk8oKQF.png)

### 架構中的三個角色

- 管理節點 Management Node  
  用於管理叢集內的其他節點，如提供配置數據、啟動並停止節點、運行備份等。由於這類節點負責管理其他節點的配置，應在啟動其他節點之前首先啟動這類節點。

- 數據節點 Data Node  
  用於保存叢集的數據，資料寫在 RAM 或 Disk。當節點有 2 個以上時就能實現集群的高可用保證，不過節點增加，集群的處理速度會變慢。

- SQL 節點 (API)  
  用來訪問叢集數據，負責 SQL 的 Table schema，提供對外應用服務。增加 API 節點會提高整個集群的並發訪問速度和整體的吞吐量，該節點可以部署在 Web 應用伺服器上，也可以部署在專用的伺服器上。

## 環境準備

搭建 MySQL Cluster 至少要一個管理節點來管理，一個 SQL 節點來實現 MySQL server 功能和兩個資料節點實現 NDB Cluster 的功能。此篇文章將測試使用雙 SQL 節點來搭建測試環境：

```
a) Management Node  10.1.5.31
b) Data Node        10.1.5.32
c) Data Node        10.1.5.33
d) SQL Node         10.1.5.32
e) SQL Node         10.1.5.33
```

以上機器使用 Ubuntu 18.04 作業系統。

## Management Node

### 下載並安裝

```
cd ~
wget https://dev.mysql.com/get/Downloads/MySQL-Cluster-8.0/mysql-cluster-community-management-server_8.0.21-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-cluster-community-management-server_8.0.21-1ubuntu18.04_amd64.deb
```

### 編輯配置檔 config.ini

```
sudo mkdir /var/lib/mysql-cluster
sudo vi /var/lib/mysql-cluster/config.ini
```

<br>

```
[ndbd default]
NoOfReplicas=2

[ndb_mgmd]
NodeId=1
hostname=10.1.5.31
datadir=/var/lib/mysql-cluster

[ndbd]
NodeId=2
hostname=10.1.5.32
datadir= /usr/local/mysql/data

[ndbd]
NodeId=3
hostname=10.1.5.33
datadir= /usr/local/mysql/data

[mysqld]
NodeId=4
hostname=10.1.5.32

[mysqld]
NodeId=5
hostname=10.1.5.33
```

配置說明如下：

```
[ndb default]
# data node 的配置
NoOfReplicas=2 # 數據節點數

[ndb_mgmd]
# 管理節點配置
hostname=10.1.5.31
datadir=/var/lib/mysql-cluster # log files 儲存目錄

[ndbd] # 數據節點配置
hostname= # Remote IP address
NodeId=2 # 為管理各節點而命名的 id，從 2 依序編碼
datadir=/usr/local/mysql/data  # 存放資料的目錄

[mysqld]
# SQL node
hostname= # Remote IP address
NodeId=3
```

### 測試是否能啟動

```
sudo ndb_mgmd -f /var/lib/mysql-cluster/config.ini
```

會出現以下訊息：

```
MySQL Cluster Management Server mysql-8.0.21 ndb-8.0.21
```

代表啟用成功。

<font color="salmon">**補充**</font>  
如果啟動後需要修改 cluster 內的 IP ，更新完 config.ini 後，啟動時改成下面指令：

```
ndb_mgmd -f /var/lib/mysql-cluster/config.ini configdir=/var/lib/mysql-cluster --reload --initial
```

### 將程序設為 service

先關掉剛剛測試的程序，

```
sudo pkill -f ndb_mgmd
```

建立服務檔，

```
sudo vi /etc/systemd/system/ndb_mgmd.service
```

貼上以下內容：

```
[Unit]
Description=MySQL NDB Cluster Management Server
After=network.target auditd.service

[Service]
Type=forking
ExecStart=/usr/sbin/ndb_mgmd -f /var/lib/mysql-cluster/config.ini --configdir=/var/lib/mysql-cluster/
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

重新載入設定檔並將服務設為開機啟動，最後啟動服務。

```
sudo systemctl daemon-reload
sudo systemctl enable ndb_mgmd
sudo systemctl start ndb_mgmd
```

查案服務狀態，

```
$sudo systemctl status ndb_mgmd
```

如果得到以下回復代表 management node 已經成功啟動。

```
． ndb_mgmd.service - MySQL NDB Cluster Management Server
   Loaded: loaded (/etc/systemd/system/ndb_mgmd.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-08-05 15:04:39 CST; 5s ago
  Process: 8445 ExecStart=/usr/sbin/ndb_mgmd -f /var/lib/mysql-cluster/config.ini --configdir=/var/lib/mysql-cluster/ (code=exited, status=0/SUCCESS)
 Main PID: 8454 (ndb_mgmd)
    Tasks: 12 (limit: 7372)
   CGroup: /system.slice/ndb_mgmd.service
           └─8454 /usr/sbin/ndb_mgmd -f /var/lib/mysql-cluster/config.ini --configdir=/var/lib/mysql-cluster/
```

或是看 1186 port 有沒有被 ndb_mgmd 占用：

```
sudo netstat -plntu
```

## Data Node

### 安裝並下載依賴程式

```
cd ~
wget https://dev.mysql.com/get/Downloads/Mysql-Cluster-8.0/mysql-cluster-community-data-node_8.0.21-1ubuntu18.04_amd64.deb
sudo apt update
sudo apt install -y libclass-methodmaker-perl
sudo dpkg -i mysql-cluster-community-data-node_8.0.21-1ubuntu18.04_amd64.deb
```

### 配置

建立 mysql 配置檔 my.cnf

```
vi /etc/my.cnf
```

貼上以下內容：

```
[mysql_cluster]
ndb-connectstring=10.1.5.31
```

其中 ndb-connectstrings 代表 cluster manager 的位址。

根據管理者的配置，建立 data node 的數據存放目錄。

```
sudo mkdir -p /usr/local/mysql/data
```

### 測試是否能啟動

```
sudo ndbd
```

會出現以下訊息，代表啟用成功。

```
2020-08-05 15:31:59 [ndbd] INFO     -- Angel connected to '10.1.5.31:1186'
2020-08-05 15:31:59 [ndbd] INFO     -- Angel allocated nodeid: 2
```

### 將程序設為 service

先關掉剛剛測試的程序，

```
sudo pkill -f ndbd
```

建立服務檔，

```
sudo vi /etc/systemd/system/ ndbd.service
```

貼上以下內容：

```
[Unit]
Description=MySQL NDB Data Node Daemon
After=network.target auditd.service

[Service]
Type=forking
ExecStart=/usr/sbin/ndbd
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

重新載入設定檔並將服務設為開機啟動，最後啟動服務。

```
sudo systemctl daemon-reload
sudo systemctl enable ndbd
sudo systemctl start ndbd
```

查案服務狀態，

```
$sudo systemctl status ndbd
```

如果得到以下回復代表 data node 已經成功啟動。

```
． ndbd.service - MySQL NDB Data Node Daemon
   Loaded: loaded (/etc/systemd/system/ndbd.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-08-05 15:47:54 CST; 7s ago
  Process: 1883 ExecStart=/usr/sbin/ndbd (code=exited, status=0/SUCCESS)
 Main PID: 1894 (ndbd)
    Tasks: 46 (limit: 7372)
   CGroup: /system.slice/ndbd.service
           ├─1894 /usr/sbin/ndbd
           └─1896 /usr/sbin/ndbd

Aug 05 15:47:54 httr2 systemd[1]: Starting MySQL NDB Data Node Daemon...
Aug 05 15:47:54 httr2 ndbd[1883]: 2020-08-05 15:47:54 [ndbd] INFO     -- Angel connected to '10.1.5.31:1186'
Aug 05 15:47:54 httr2 ndbd[1883]: 2020-08-05 15:47:54 [ndbd] INFO     -- Angel allocated nodeid: 2
Aug 05 15:47:54 httr2 systemd[1]: Started MySQL NDB Data Node Daemon.
```

看 netstat 中是否有被 ndbd 進程占用的 port。

```
sudo netstat -plntu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 10.1.5.32:35499          0.0.0.0:*               LISTEN      24377/ndbd
tcp        0      0 10.1.5.32:35189          0.0.0.0:*               LISTEN      24377/ndbd
```

## SQL Node

### 下載

因為也是跟 data node 一樣把壓縮檔放在家目錄下，所以另外建一個資料夾 install 存放 SQL Node 的安裝檔。

```
cd ~
wget https://dev.mysql.com/get/Downloads/Mysql-Cluster-8.0/mysql-cluster_8.0.21-1ubuntu18.04_amd64.deb-bundle.tar
mkdir install
tar -xvf mysql-cluster_8.0.21-1ubuntu18.04_amd64.deb-bundle.tar -C install/
cd install
```

### 下載依賴包

```
sudo apt update
sudo apt install -y libaio1 libmecab2
```

### 開始安裝 MySQL Cluster dependencies

```
sudo dpkg -i mysql-common_8.0.21-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-cluster-community-client-core_8.0.21-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-cluster-community-client_8.0.21-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-client_8.0.21-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-cluster-community-server-core_8.0.21-1ubuntu18.04_amd64.deb
sudo dpkg -i mysql-cluster-community-server_8.0.21-1ubuntu18.04_amd64.deb
```

在安裝 mysql-cluster-community-server 的過程中，會要求輸入 mysql root 的密碼。

![](https://imgur.com/Cyu2sP9.png)

之後會詢問是否要使用 mysql 8.0 提供的新功能加強密碼加密，

![](https://imgur.com/Qw2RtaH.png)

這邊選擇保持原本的密碼方式， 兼容性較高。（此設置在安裝完畢後可依需求改變）

![](https://imgur.com/7s7gi8C.png)

安裝 MySQL server binary。

```
sudo dpkg -i mysql-server_8.0.21-1ubuntu18.04_amd64.deb
```

### 重配置 my.cnf 檔案

```
sudo vi /etc/mysql/my.cnf
```

在末端加上以下內容：

```
[mysqld]
ndbcluster

[mysql_cluster]
ndb-connectstring=10.1.5.31
bind-address= 0.0.0.0
```

配置檔內容說明如下：

```
[mysqld]
# mysqld 程序的配置
ndbcluster
# 執行 ndb 儲存引擎

[mysql_cluster]
ndb-connectstring=10.1.5.31
# management node 位址
bind-address= 0.0.0.0
# 代表可提供外部網路連線 remote 管理，若沒有此參數的話，SQL 只能在 local 端使用
```

### 重啟 mysql service 套用以上變更

```
sudo systemctl restart mysql
```

加入開機啟動。

```
sudo systemctl enable mysql
```

## 驗證集群安裝

進入 `SQL node` 所在的機器開啟 `ndb_mgm` console 連進 management node。輸入 `show`，可列出 cluster 的整體架構。

```
ndb_mgm
> show
```

```
ndb_mgm
-- NDB Cluster -- Management Client --
ndb_mgm> show
Connected to Management Server at: 10.1.5.31:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=2    @10.1.5.32  (mysql-8.0.21 ndb-8.0.21, Nodegroup: 0)
id=3    @10.1.5.33  (mysql-8.0.21 ndb-8.0.21, Nodegroup: 0, *)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @10.1.5.31  (mysql-8.0.21 ndb-8.0.21)

[mysqld(API)]   2 node(s)
id=4    @10.1.5.32  (mysql-8.0.21 ndb-8.0.21)
id=5    @10.1.5.33  (mysql-8.0.21 ndb-8.0.21)

ndb_mgm> 2 STATUS
Node 2: started (mysql-8.0.21 ndb-8.0.21)

ndb_mgm>
```

SQL node 所在的機器開啟 MySQL Server，

```
mysql -u root -p
```

進入 mysql 後，下下面指令：

```
> SHOW ENGINE NDB STATUS \G
*************************** 1. row ***************************
  Type: ndbclus
  Name: connection
Status: cluster_node_id=4, connected_host=10.1.5.31, connected_port=1186, number_of_data_nodes=2, number_of_ready_da                                                                           ta_nodes=2, connect_count=0
*************************** 2. row ***************************
  Type: ndbclus
  Name: NdbTransaction
Status: created=2, free=2, sizeof=392
*************************** 3. row ***************************
  Type: ndbclus
  Name: NdbOperation
Status: created=4, free=4, sizeof=944
...
...
```

表示成功連到 mysql cluster。

## 使用遠端 workblench 進行連線

先進入 SQL node 在本地端新建一個使用者並賦予存取所有位置的權限：

```
mysql -u root -p
```

<br>

```
mysql > CREATE USER 'user'@'%' IDENTIFIED BY 'password';
mysql > GRANT ALL ON *.* TO 'user'@'%';
```

開啟 MySQLworkblench 新增連線對象為 cluster 內任何一個 SQL node，便可成功連線。

### 建立資料表

建立資料表時須指定使用 ndbcluster 引擎，才可多點同步。

![](https://imgur.com/yekW0XQ.png)

### 確認

寫入資料，

![](https://imgur.com/eE3gWj4.png)

進入另一台 SQL node 查看資料是否有同步。

![](https://imgur.com/JfkmpZD.png)
