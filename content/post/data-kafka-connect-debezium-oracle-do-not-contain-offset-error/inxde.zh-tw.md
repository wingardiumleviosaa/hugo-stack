---
title: "[Kafka] Debezium Oracle Connect: Online REDO LOG files or archive log files do not contain the offset"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2024-03-16 14:35:00
slug: "kafka-connect-debezium-oracle-do-not-contain-offset-error"
---

### 問題

Debezium Oracle sink connector 跑了一段時間後發生以下錯誤：

```bash
ERROR Mining session stopped due to the {} (io.debezium.connector.oracle.logminer.LogMinerHelper)
io.debezium.DebeziumException: Online REDO LOG files or archive log files do not contain the offset scn 7470041329489.  Please perform a new snapshot.
```

<!--more-->

### 解決

方法參考[官網](https://debezium.io/documentation/faq/#how_to_remove_committed_offsets_for_a_connector)，步驟如下：

1. 查看目前 storage offset 的 topic

```bash
kafkacat -b 172.22.51.207:8093,172.22.51.207:8094,172.22.51.207:8095 -C -t hq_connect_offsets -f 'Partition(%p) %k %s\n'
```

2. pause / stop the connector

3. 刪除 offset

```bash
echo '["source-mes-to-kafka-initial",{"server":"MES-INFO"}]|' | kafkacat -P -Z -b 172.22.51.207:8093,172.22.51.207:8094,172.22.51.207:8095 -t hq_connect_offsets -K \| -p 3
```

4. 重啟 kafka connect (resume & restart task)，就正常了。

### 優化

觀察一陣子後發現使用 `schema_mode = initial` 的 kafka debezium oracle source connector，較常發生這個錯誤，推測因為 offset 紀錄的 SCN 超過兩天的保存期限而讀取不到下一個紀錄點導致 connector failed。參考[官網 low-change-frequency-offset-management](https://debezium.io/documentation/reference/2.4/connectors/oracle.html#low-change-frequency-offset-management)，須將 heartbeat.interval.ms 設定項啟用，目前為五分鐘 sync 一次。
