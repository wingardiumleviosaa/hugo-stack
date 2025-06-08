---
title: "[Oracle] Install Oracle 11g XE and Establish CDC by Debezium"
author: Ula
tags:
  - Debezium
  - Oracle
categories:
  - DataEngineering
  - ApacheKafka
date: 2022-01-19 11:03:00
slug: "install-oracle-11g-and-establish-cdc-by-debezium"
---

## Debezium

### 簡介

Debezium 是一個由 RedHat 開源的基於資料庫變更日誌的實時變更數據捕獲（CDC）工具，構建在 Apache Kafka 之上，是 Apache Kafka Connect 的 Source Connector，可以實時獲取行級別（row-level）資料的更改事件（INSERT、UPDATE 和 DELETE）並同步到 Kafka。目前支援的常見資料庫有 MySQL(binlog)、Oracle(logminer or xstream)、MongoDB(change streams)、PostgreSQL(logical replication stream mode)、SQL Server(transaction log)...等。本文範例是使用 Oracle 的 logminer 日誌透過 Debezium 獲取指定資料庫的變更事件。

<!--more-->

### CDC

CDC 全稱是 Change Data Capture 變更數據捕獲，它是一個比較廣義的概念，只要能捕獲變更的資料，都可以稱為 CDC，主要有基於查詢的 CDC 和基於日誌的 CDC。

|                                |                           基於查詢的 CDC                           |                       基於日誌的 CDC                       |
| :----------------------------: | :----------------------------------------------------------------: | :--------------------------------------------------------: |
|              概念              | 每次捕獲變更發起 Select 查詢進行全表掃描，過濾出查詢之間變更的資料 | 讀取資料儲存系統的 log ，例如 Mysql 裡面的 binlog 持續監控 |
|            開源產品            |                      Sqoop, Kafka JDBC Source                      |                  Canal, Maxwell, Debezium                  |
|            執行模式            |                               Batch                                |                         Streaming                          |
|       捕獲所有資料的變化       |                                 X                                  |                             O                              |
|    低延遲，不增加資料庫負載    |                                 X                                  |                             O                              |
| 不侵入業務（LastUpdated 欄位） |                                 X                                  |                             O                              |
|   捕獲刪除事件和舊記錄的狀態   |                                 X                                  |                             O                              |
|        捕獲舊記錄的狀態        |                                 X                                  |                             O                              |

## 安裝 Oracle 11g Express Edition

### Create Container

```
docker run -d -it --name oracle -p 1521:1521 -e ORACLE_ALLOW_REMOTE=true -v oracle:/u01/app/oracle --restart=always wnameless/oracle-xe-11g-r2
```

### Connect to the Database

```bash
docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED         STATUS         PORTS                                                         NAMES
624e811e8e0b   wnameless/oracle-xe-11g-r2   "/bin/sh -c '/usr/sb…"   3 minutes ago   Up 3 minutes   22/tcp, 8080/tcp, 0.0.0.0:1521->1521/tcp, :::1521->1521/tcp   oracle

docker logs -f oracle
Starting Oracle Net Listener.
Starting Oracle Database 11g Express Edition instance.
```

使用資料庫工具連線進入

```
hostname: localhost
port: 1521
sid: xe
username: system
password: oracle
```

Password for SYS & SYSTEM

```
oracle
```

![](https://imgur.com/HLKyKzq.png)

![](https://imgur.com/pDC97QK.png)

## Oracle CDC (Logmior) Configuration

### 開啟日誌歸檔

1. 進入容器並切換成 oracle 使用者，再透過 sqlplus 以 sys 用戶登入進入資料庫

```
docker exec -it oracle /bin/bash
[root@b2a1bd25aa97 /]# su — oracle
[root@b2a1bd25aa97 /]# sqlplus sys/oracle as sysdba
```

2. 檢查日誌歸檔是否已開啟。

```sql
archive log list;
```

- 若顯示 “Database log mode: No Archive Mode”，說明日誌歸檔未開啟，繼續執行下一步。
- 若顯示 “Database log mode: Archive Mode”，說明日誌歸檔已開啟，直接跳到退出數據庫連接。

3. 執行以下命令配置歸檔日誌參數。

```sql
alter system set db_recovery_file_dest_size = 10G;
alter system set db_recovery_file_dest = '/u01/app/oracle/oradata/recovery_area' scope=spfile;
```

其中：

- 10G 為日誌文件存儲空間的大小，請根據實際情況設置。
- /u01/app/oracle/oradata/recovery_area 為日誌存儲路徑，須確保路徑提前創建。

4. 開啟日誌歸檔。
   {{< notice info >}}
   須知：
   開啟日誌歸檔功能需重啟數據庫，重啟期間將導致業務中斷，請謹慎操作。
   歸檔日誌會佔用較多的磁盤空間，若磁盤空間滿了會影響業務，請定期清理過期歸檔日誌。
   {{< /notice >}}
   </br>

```sql
shutdown immediate;
startup mount;
alter database archivelog;
alter database open;
```

5. 確認日誌歸檔是否已成功開啟。

```sql
archive log list;
```

當顯示 “Database log mode: Archive Mode”，說明日誌歸檔已開啟。

6. 退出資料庫連接。

```sql
exit;
```

![](https://imgur.com/Ghsl3lu.png)

### 安裝 logminer 工具

1. 重新連接到資料庫

```sql
sqlplus sys/oracle as sysdba
# sqlplus sys/password@host:port/SID as sysdba
```

2. 檢查 LogMiner 工具是否已安裝
   如果有返回訊息說明 LogMiner 已安裝

```sql
desc DBMS_LOGMNR
desc DBMS_LOGMNR_D
```

![](https://imgur.com/cFw9dsR.png)

如果沒有訊息返回則需要另外安裝，請執行以下命令

```sql
-- 創建 dbms_logmnr 包，用來分析歸檔日誌
@$ORACLE_HOME/rdbms/admin/dbmslm.sql

-- 創建DBMS_LOGMNR_D包，該包用來創建數據字典文件。
@$ORACLE_HOME/rdbms/admin/dbmslmd.sql
```

### 啟動最小附加日誌

啟用附加最小日誌（supplemental_log_data_min）可以確保 LogMiner（或其他任何基於 LogMiner 的產品）可以支援行連結、簇表、索引組織表等，避免在挖掘時會出現信息遺漏的情況。
驗證是否有啟用附加日誌：

```sql
SELECT supplemental_log_data_min, supplemental_log_data_pk, supplemental_log_data_all FROM v$database;
```

預設是沒有開啟的(return no/no/no)，在 **資料庫級別** 啟用最小補充日誌記錄，按如下方式配置：

```sql
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (PRIMARY KEY) COLUMNS;
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
# commit the changes
ALTER SYSTEM ARCHIVE LOG CURRENT;
```

![](https://imgur.com/ScoREMo.png)

如果只是想給某個指定的表，開啟附加日誌記錄，參考下面。

```sql
ALTER TABLE HR.EMPLOYEES ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS;
```

### 建立 logminer 用戶並賦予權限

1. 創建專屬 tablespace

```sql
CREATE TABLESPACE LOGMINER_TBS DATAFILE '/u01/app/oracle/oradata/XE/logminer_tbs.dbf' SIZE 25M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
CREATE USER logminer IDENTIFIED BY logminer DEFAULT TABLESPACE logminer_tbs QUOTA UNLIMITED ON logminer_tbs;
```

2. 創建 logMiner 使用者並配置權限。

```sql
grant create session, alter session, alter system, execute_catalog_role, select_catalog_role, select any transaction, select any dictionary, select any table, create table, alter any table, create sequence, lock any table, flashback any table, create tablespace, drop tablespace to logminer;
grant connect to logminer;
grant trigger to logminer;
grant select on SYSTEM.LOGMNR_COLto logminer;
grant select on SYSTEM.LOGMNR_OBJto logminer;
grant select on SYSTEM.LOGMNR_USERto logminer;
grant select on SYSTEM.LOGMNR_UIDto logminer;
grant execute on SYS.DBMS_LOGMNR to logminer;
grant execute on SYS.DBMS_LOGMNR_D to logminer;
grant execute on SYS.DBMS_LOGMNR_LOGREP_DICT to logminer;
grant execute on SYS.DBMS_LOGMNR_SESSION to logminer;
grant select on V_$DATABASE to logminer;
GRANT SELECT ON V_$LOG TO logminer;
GRANT SELECT ON V_$LOG_HISTORY TO logminer;
GRANT SELECT ON V_$LOGMNR_LOGS TO logminer;
GRANT SELECT ON V_$LOGMNR_CONTENTS TO logminer;
GRANT SELECT ON V_$LOGMNR_PARAMETERS TO logminer;
GRANT SELECT ON V_$LOGFILE TO logminer;
GRANT SELECT ON V_$ARCHIVED_LOG TO logminer;
GRANT SELECT ON V_$ARCHIVE_DEST_STATUS TO logminer;
GRANT select on gv_$logmnr_parameters to logminer;
GRANT select on gv_$logmnr_logs to logminer;
GRANT select on gv_$archived_log to logminer;
grant select on v_$logmnr_dictionary to logminer;
grant select on V_$TRANSACTION to logminer;
GRANT EXECUTE ON DBMS_LOGMNR TO logminer;
GRANT EXECUTE ON DBMS_LOGMNR_D TO logminer;
grant all on DBMS_LOGMNR_D to logminer;
grant all on DBMS_LOGMNR to logminer;
grant select on dba_objects to logminer;
GRANT UNLIMITED TABLESPACE TO logminer;
```

## 架設 Debezium 環境

### 安裝 Zookeeper ＆ Kafka

```sh
docker run -d -it --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.5

# 為了給外部連線使用，kafka 需設置 advertised_listeners 參數
docker run -d -it --name kafka -p 9092:9092 -e ADVERTISED_HOST_NAME=10.13.1.105 -e ADVERTISED_LISTENERS=PLAINTEXT://10.13.1.105:9092 --link zookeeper:zookeeper debezium/kafka:1.5
```

### 準備新的 Kafka Connect Image

根據官網的提示，如果要連接 oracle 需要自行下載並複製 jdbc driver 到 kafka connect 中。

> Due to licensing requirements, the Debezium Oracle connector does not ship with the Oracle JDBC driver or XStream API files. You must download these files directly from Oracle and add them to your environment.

步驟如下：

1. 下載 oracle instant clinet

```bash
wget https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-basic-linux.x64-18.5.0.0.0dbru.zip
# 解壓縮
unzip instantclient-basic-linux.x64-18.5.0.0.0dbru.zip
```

2. 啟動臨時的 container

```bash
docker run -d -it --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=test_connect_configs -e OFFSET_STORAGE_TOPIC=test_connect_offsets -e STATUS_STORAGE_TOPIC=test_connect_statuses --link zookeeper:zookeeper --link kafka:kafka debezium/connect:1.5
```

其中，需要傳入如下環境變數：

- GROUP_ID：若需要啟動多個 Debezium 例項組成叢集，那麼它們的 GROUP_ID 必須被設一樣。
- CONFIG_STORAGE_TOPIC：指定用來存 connector 的 config 資訊的 kafka topic。
- STATUS_STORAGE_TOPIC：指定用來儲存 connector 的狀態資訊的 kafka topic。
- OFFSET_STORAGE_TOPIC：只定用來存 connector 監控資料流的 offset 的 kafka topic，若使用的是 Oracle Connector，那麼該 topic 存的就是 Oracle logminer 的 scn。

3. 製作新的 kafka connect image
   首先複製整個 instant clinet 的資料夾到 container 中

```
docker cp ./instantclient_18_5/ connect:/kafka/
```

進入 container 中，將 instantclinet 下的 jdbc driver 複製到 /kafka/libs 底下 (因為本文範例使用 logminer 監聽 CDC，故僅需將 ojdbc8.jar 複製，xstreams.jar 檔只有在 xstream 模式下才需要使用到)

```
docker exec -it connect /bin/bash
[kafka@88c5e1f0af86 ~]ls
LICENSE  bin     config.orig  data           instantclient_18_5  logs
NOTICE   config  connect      external_libs  libs
[kafka@88c5e1f0af86 ~]cp instantclient_18_5/ojdbc8.jar libs/

```

退出 container，查看 container id 後將狀態 commit 成新的 image

```
docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED        STATUS        PORTS                                                                                                                                                 NAMES
88c5e1f0af86   connect:1.5              "/docker-entrypoint.…"   21 hours ago   Up 21 hours   8778/tcp, 9092/tcp, 0.0.0.0:8083->8083/tcp, :::8083->8083/tcp, 9779/tcp                                                                               connect
```

</br>

```
docker commit 88c5e1f0af86 connect:tmp
```

準備一個 Dockerfile

```docker
FROM connect:tmp
USER root
RUN yum -y install libaio && yum clean all
USER kafka
```

Build 最終的 docker image

```sh
docker build -t connect:1.5.1
```

### 安裝 Kafka Connect

從上一步驟包的 kafka connect image 建立正式的 connect container，注意要加上 `LD_LIBRARY_PATH` 環境變數指向 instantclient 的目錄

```bash
docker run -d -it --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=test_connect_configs -e OFFSET_STORAGE_TOPIC=test_connect_offsets -e STATUS_STORAGE_TOPIC=test_connect_statuses -e LD_LIBRARY_PATH=/kafka/instantclient_18_5 --link zookeeper:zookeeper --link kafka:kafka connect:1.5.1
```

查看 kafka connect 的資訊以及 connector 列表

```bash
curl -H "Accept:application/json" localhost:8083/
curl -H "Accept:application/json" localhost:8083/connectors/
```

### 配置 Oracle Connector

呼叫 kafka connect 的 connector API 以創建 connector

```bash
curl -i -X PUT -H "Accept:application/json" -H "Content-Type:application/json"  localhost:8083/connectors/oracle1/config -d '{"name": "oracle1", "connector.class": "io.debezium.connector.oracle.OracleConnector", "tasks.max": "1","database.server.name": "XE", "database.hostname": "10.13.1.105", "database.port": "1521", "database.user": "logminer", "database.password": "logminer", "database.dbname": "XE", "database.out.server.name": "dbzout", "database.schema": "HR", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "utest", "database.connection.adapter": "logminer", "table.include.list": "HR.*", "snapshot.mode": "initial", "database.tablename.case.insensitive": "true", "decimal.handling.mode": "string", "log.mining.strategy": "online_catalog"}'
```

覺得 curl 不方便，可以換用 postman：

```json
{
  "name": "oracle1",
  "connector.class": "io.debezium.connector.oracle.OracleConnector",
  "tasks.max": "1",
  "database.hostname": "10.13.1.105",
  "database.port": "1521",
  "database.user": "logminer",
  "database.password": "logminer",
  "database.dbname": "XE",
  "database.server.name": "XE",
  "database.history.kafka.bootstrap.servers": "kafka:9092",
  "database.history.kafka.topic": "utest",
  "database.connection.adapter": "logminer",
  "table.include.list": "HR.*",
  "log.mining.strategy": "online_catalog"
}
```

![](https://imgur.com/GrPflZL.png)

其中：

- name：註冊到 Kafka Connect 服務的 Connector 名稱。
- database.server.name：虛擬的資料庫 Server 名稱，可以根據實際需求定義，定義 Kafka topic 時會使用該值。
- table.include.list：監聽的數據表列表，以 `,` 分割。`HR.*` 表示監聽 HR 下的所有資料表。
- 每個被監控的表在 Kafka 都會對應一個 topic，topic 的命名規範是 `<database.server.name>.<schema>.<table>`
- log.mining.strategy：預設為 redo_log_catalog，會造成 kafka consume 延遲，查官網後發現，如果要頻繁修改 DDL (如表的 schema) 才需要設置。使用 `online_catalog` 只監聽資料表內的 data 變化不會額外寫表的 DDL 資訊，能加速 logminer！

### 查看 Connector 狀態

下 GET `/connectors/<connectorName>/status` 的 api

```oracle1
curl -H "Accept:application/json" http://127.0.0.1:8083/connectors/oracle1/status
```

如果成功會回傳以下回覆

```json
{
  "name": "oracle1",
  "connector": {
    "state": "RUNNING",
    "worker_id": "172.17.0.5:8083"
  },
  "tasks": [
    {
      "id": 0,
      "state": "RUNNING",
      "worker_id": "172.17.0.5:8083"
    }
  ],
  "type": "source"
}
```

### 增刪改資料表並查看 Kafka Topic 是否有收到 log 資料

使用 kafka clinet 腳本程式列出目前 kafka server 上的所有 topic

```bash
./kafka-topics.sh --list --bootstrap-server 10.13.1.105:9092
XE
XE.HR.COUNTRIES
XE.HR.DEPARTMENTS
XE.HR.EMPLOYEES
XE.HR.JOBS
XE.HR.JOB_HISTORY
XE.HR.LOCATIONS
XE.HR.REGIONS
__consumer_offsets
test_connect_configs
test_connect_offsets
test_connect_statuses
utest
```

開啟 kafka consumer 消費 logminer 捕捉 HR.EMPLOYEES 表的變化

```bash
./kafka-console-consumer.sh --bootstrap-server 10.13.1.105:9092 --topic XE.HR.EMPLOYEES --from-beginning
```

進入資料庫操作目標資料表 HR.EMPLOYEES，新增一筆員工資料

```sql
sqlplus sys/oracle@127.0.0.1:1521/xe as sysdba
insert into HR.EMPLOYEES(EMPLOYEE_ID,FIRST_NAME,LAST_NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,JOB_ID,SALARY,MANAGER_ID,DEPARTMENT_ID) values(206,'Kawhi','Leonard','KAWHI','111.222.3333',2022-01-19 00:00:00,'SH_CLERK',15000,124,50);
```

回到 kafka consumer 的 terminal 可以發現可以收到 logminer 變動的資料

![](https://imgur.com/00Ul63W.png)

![](https://imgur.com/KArHEsu.png)

## Reference

- https://www.796t.com/article.php?id=169153
- https://www.itread01.com/content/1549275121.html
- https://debezium.io/documentation/reference/stable/connectors/oracle.html
- https://python.iitter.com/other/229206.html
