---
title: "[Kafka] ksqlDB 流主題無過濾資料，但卻跟源主題有數量落差原因排查"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2024-03-16 14:10:00
slug: "ksqldb-missing-data-debug"
---

使用 ksqlDB 宣告了兩個 stream，第一個 stream 為現有的 topic 的源資料流，並重新定義 kafka message 的 event time，另一個是基於原生資料流派生的 stream，主要只差在更改 partition 數量以利後續跟其他不同 partition 數量的流做 join。

<!-- more -->

```sql
-- 定義原生流給源主題
CREATE STREAM IF NOT EXISTS NODERED_CPPS (
    MACHINE_CODE STRING,
    ts BIGINT,
    asset_id STRING,
    `values` STRING
) WITH (
    CLEANUP_POLICY = 'delete',
    KAFKA_TOPIC = 'nodered-cpps',
    KEY_FORMAT = 'JSON',
    VALUE_FORMAT = 'JSON',
    TIMESTAMP = 'TS'
);

-- 新增一條派生流，使用一致的 partition number，才可以被相同 partition 的 steam 做 join
CREATE STREAM IF NOT EXISTS NEW_NODERED_CPPS
WITH (
	KAFKA_TOPIC='NEW_NODERED_CPPS',
	PARTITIONS=6,
	REPLICAS=1,
	RETENTION_MS=864000000,
	KEY_FORMAT='JSON',
	VALUE_FORMAT='JSON'
) AS SELECT * FROM NODERED_CPPS EMIT CHANGES;
```

按照正常情況來說，因為派生流沒有做任何過濾，最終的訊息數量應該要跟原生的 topic 一樣，但部署後兩天查看兩個 topic，派生流的訊息整整少了原生流的三分之一。查看 ksqlDB server 的訊息後發現問題如下：

## 超過 kafka client request size

ksqlDB server 收到的錯誤訊息如下：

```perl
[2024-01-26 01:00:47,941] INFO Restarting query CSAS_NEW_NODERED_CPPS_1343 thread _confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54-StreamThread-2 (attempt #116) (io.confluent.ksql.util.QueryMetadataImpl:495)
[2024-01-26 01:00:47,941] ERROR {"type":4,"deserializationError":null,"recordProcessingError":null,"productionError":null,"serializationError":null,"kafkaStreamsThreadError":{"errorMessage":"Unhandled exception caught in streams thread","threadName":"_confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54-StreamThread-2","cause":["Error encountered sending record to topic NEW_NODERED_CPPS for task 0_2 due to:\norg.apache.kafka.common.errors.RecordTooLargeException: The message is 1183085 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration.\nException handler choose to FAIL the processing, no more records would be sent.","The message is 1183085 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration."]}} (processing.CSAS_NEW_NODERED_CPPS_1343.ksql.logger.thread.exception.uncaught:44)
[2024-01-26 01:00:47,941] ERROR stream-client [_confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54] Replacing thread in the streams uncaught exception handler (org.apache.kafka.streams.KafkaStreams:530)
org.apache.kafka.streams.errors.StreamsException: Error encountered sending record to topic NEW_NODERED_CPPS for task 0_2 due to:
org.apache.kafka.common.errors.RecordTooLargeException: The message is 1183085 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration.
Exception handler choose to FAIL the processing, no more records would be sent.
	at org.apache.kafka.streams.processor.internals.RecordCollectorImpl.recordSendError(RecordCollectorImpl.java:284)
	at org.apache.kafka.streams.processor.internals.RecordCollectorImpl.lambda$send$1(RecordCollectorImpl.java:254)
	at org.apache.kafka.clients.producer.KafkaProducer.doSend(KafkaProducer.java:1077)
...
Caused by: org.apache.kafka.common.errors.RecordTooLargeException: The message is 1183085 bytes when serialized which is larger than 1048576, which is the value of the max.request.size configuration.
[2024-01-26 01:00:47,941] INFO stream-thread [_confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54-StreamThread-2] Informed to shut down (org.apache.kafka.streams.processor.internals.StreamThread:1168)
[2024-01-26 01:00:47,941] INFO stream-thread [_confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54-StreamThread-2] State transition from RUNNING to PENDING_SHUTDOWN (org.apache.kafka.streams.processor.internals.StreamThread:239)
[2024-01-26 01:00:47,941] INFO stream-client [_confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54] Adding StreamThread-3, there will now be 4 live threads and the new cache size per thread is 2500000 (org.apache.kafka.streams.KafkaStreams:1029)
[2024-01-26 01:00:47,941] INFO stream-thread [_confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54-StreamThread-3] Creating restore consumer client (org.apache.kafka.streams.processor.internals.StreamThread:356)
[2024-01-26 01:00:47,942] INFO ConsumerConfig values:
	allow.auto.create.topics = true
	auto.commit.interval.ms = 5000
	auto.include.jmx.reporter = true
	auto.offset.reset = none
	bootstrap.servers = [kafka-headless.platform.svc.cluster.local:9093]
	check.crcs = true
	client.dns.lookup = use_all_dns_ips
	client.id = _confluent-ksql-ksql_dev_ops_query_CSAS_NEW_NODERED_CPPS_1343-8ae61fa4-f6d2-4b36-ba53-7cd24fd34c54-StreamThread-3-restore-consumer
	....
```

原來是單筆訊息超過 Kafka Producer 的指定大小，Producer 能發送訊息的最大值預設為 1048576B，1 MB。在觸發錯誤的時候會重啟 Kafka client，也就是 ksqlDB 的 CTAS。導致訊息遺失的也不僅僅是訊息過大的這條，而是產生錯誤後的到 client 復原時中間的好幾筆資料!!!

## 修改 KSQLDB 參數

producer 指定訊息大小的參數為 `max.request.size` ，要注意的是該參數涉及 broker 端的 `message.max.bytes` 參數的聯動，如果 broker 的 `message.max.bytes` 參數設定為 10 MB，而 `max.request.size` 設定為 20 MB，當傳送一條大小為 15 MB 的訊息時，producer 還是會報錯。因此需要連動修改兩個參數，以防錯誤，修改如下：

- message.max.bytes （在 broker 端修改）
- max.request.size （在客戶端修改）

那 ksqlDB 有關 stream 的訊息大小的參數為 `ksql.streams.producer.max.request.size`，看是使用哪種方式部署，更新該參數到 ksqlDB 的 properties 檔案，調整完 size 之後就成功了!!!!!!!!!!!!!!! 讚

![](https://imgur.com/OZEZsDo.png)
