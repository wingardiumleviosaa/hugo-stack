---
title: Debezium 1.5 snapshot.mode schema_only not work
tags:
  - Debezium
  - Kafka
  - Oracle
categories:
  - DataEngineering
  - ApacheKafka
date: 2022-03-22 14:21:00
slug: debezium-snapshot-mode-schema-only-not-work
---

使用 Debezium 1.5 連上 connector 後，發現資料會從資料表的最舊資料一筆資料開始送，查官網後發現有 `snapshot.mode` 的參數可以設置，但設置完畢後卻發現有 error，本篇記錄解法。

<!--more-->

## snapshot.mode

Debezium 1.5 版本的 Oracle connector 的配置中，用 snapshot.mode 配置項表示快照模式。默認為 `initial` 模式，當連接器啟動的時候，會執行一次數據庫初始的一致性快照任務。
第二種快照模式是 `schema_only`。在這種情況下，connector 仍會捕獲相關表的結構，但它不會在啟動時產生 READ 事件創建完整數據集。故如果只對從現在開始的數據更改感興趣，而不對所有記錄的完整當前狀態感興趣，須將模式改為 `schema_only`。

## 錯誤

在 connector 的設定項加上 `"snapshot.mode": "schema_only"` 後，發現 connector 會報以下錯誤：

```
2022-03-22 05:06:45,029 ERROR  ||  WorkerSourceTask{id=oracle2-0} Task threw an uncaught and unrecoverable exception. Task i                              s being killed and will not recover until manually restarted   [org.apache.kafka.connect.runtime.WorkerTask]
org.apache.kafka.connect.errors.ConnectException: An exception occurred in the change event producer. This connector will be                               stopped.
        at io.debezium.pipeline.ErrorHandler.setProducerThrowable(ErrorHandler.java:42)
        at io.debezium.connector.oracle.logminer.LogMinerStreamingChangeEventSource.execute(LogMinerStreamingChangeEventSour                              ce.java:208)
        at io.debezium.pipeline.ChangeEventSourceCoordinator.streamEvents(ChangeEventSourceCoordinator.java:152)
        at io.debezium.pipeline.ChangeEventSourceCoordinator.lambda$start$0(ChangeEventSourceCoordinator.java:119)
        at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
        at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
        at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
        at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
        at java.base/java.lang.Thread.run(Thread.java:834)
Caused by: io.debezium.DebeziumException: Supplemental logging not configured for table EMESHY.EMESP.tp_sn_log.  Use command                              : ALTER TABLE EMESP.tp_sn_log ADD SUPPLEMENTAL LOG DATA (ALL) COLUMNS
        at io.debezium.connector.oracle.logminer.LogMinerHelper.checkSupplementalLogging(LogMinerHelper.java:407)
        at io.debezium.connector.oracle.logminer.LogMinerStreamingChangeEventSource.execute(LogMinerStreamingChangeEventSour                              ce.java:132)
        ... 7 more
```

## 錯誤排查

按照錯誤原因是指定的表沒有設置 Supplemental Log，但理應不可能，因為在預設的 snapshot mode initial 時是可連接成功的，還是按照指示進入資料庫設置，

![](https://imgur.com/2UEJK3g.png)

然結果不出所料的回報已經設置了，使用網路上的另一個 command 查看 `ALL_LOG_GROUP` 的配置：

![](https://imgur.com/KWFKB7v.png)

一樣是顯示在我指定的 TP_SN_LOG 表有設置補充日誌。
無意間在 community 發現也有人遇到一樣的錯誤([gitter thread](https://gitter.im/debezium/user?at=6034b854e634904e60ba19a4))，發現可能是 bug，其實可以看到錯誤訊息的內容把原本應該大寫的表名 TP_SN_LOG 轉成小寫 tp_sn_log。查看 [jira issue](https://issues.redhat.com/browse/DBZ-3190) 提到，照理來說 oracle 標識符在查詢中不區分大小寫，除非它們在字符串謂詞中顯式使用，即 “WHERE TABLE_NAME = 'table'”，或者標識符是用雙引號創建的。但我在送 config 時就是使用大寫押~~怪!

### 解決方式

暫時在 connector 設定項加上 `database.tablename.case.insensitive` 參數，值設為 `false`，即可順利解決。超怪 =\_\_\_=，**非**大小寫不敏感不就是指敏感的意思嘛!!! 只能當作是 1.5 的 bug 了。

附上最終的 config

```json
{
  "name": "oracle2",
  "connector.class": "io.debezium.connector.oracle.OracleConnector",
  "tasks.max": "1",
  "database.hostname": "10.90.1.207",
  "database.port": "1521",
  "database.user": "logminer",
  "database.password": "logminer",
  "database.dbname": "EMESHY",
  "database.server.name": "oracle",
  "database.history.kafka.bootstrap.servers": "kafka:9092",
  "database.history.kafka.topic": "tpsn2",
  "database.connection.adapter": "logminer",
  "table.include.list": "EMESP.TP_SN_LOG",
  "log.mining.strategy": "online_catalog",
  "snapshot.mode": "schema_only",
  "database.tablename.case.insensitive": "false"
}
```

## Reference

- https://debezium.io/documentation/reference/1.5/connectors/oracle.html#oracle-snapshots
- https://gitter.im/debezium/user?at=6034b854e634904e60ba19a4
