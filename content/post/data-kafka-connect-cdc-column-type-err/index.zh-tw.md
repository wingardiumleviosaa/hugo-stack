---
title: "[Kafka] CDC Connect Oracle -> Postgres VariableScaleDecimal Error"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2024-03-16 14:10:00
slug: "kafka-cdc-column-type-err"
---

### 錯誤

使用 kafka connector 部署 postgresql 的 sink connector 抓取 Oracle 的 CDC 資料時，遇到 `io.debezium.data.VariableScaleDecimal (STRUCT) type doesn't have a mapping to the SQL database column type` 的錯誤。

<!--more-->

```c
Caused by: org.apache.kafka.connect.errors.ConnectException: io.debezium.data.VariableScaleDecimal (STRUCT) type doesn't have a mapping to the SQL database column type
        at io.confluent.connect.jdbc.dialect.GenericDatabaseDialect.getSqlType(GenericDatabaseDialect.java:1948)
        at io.confluent.connect.jdbc.dialect.PostgreSqlDatabaseDialect.getSqlType(PostgreSqlDatabaseDialect.java:340)
        at io.confluent.connect.jdbc.dialect.GenericDatabaseDialect.writeColumnSpec(GenericDatabaseDialect.java:1864)
        at io.confluent.connect.jdbc.dialect.GenericDatabaseDialect.lambda$writeColumnsSpec$39(GenericDatabaseDialect.java:1853)
        at io.confluent.connect.jdbc.util.ExpressionBuilder.append(ExpressionBuilder.java:560)
        at io.confluent.connect.jdbc.util.ExpressionBuilder$BasicListBuilder.of(ExpressionBuilder.java:599)
        at io.confluent.connect.jdbc.dialect.GenericDatabaseDialect.writeColumnsSpec(GenericDatabaseDialect.java:1855)
        at io.confluent.connect.jdbc.dialect.GenericDatabaseDialect.buildCreateTableStatement(GenericDatabaseDialect.java:1772)
        at io.confluent.connect.jdbc.sink.DbStructure.create(DbStructure.java:121)
        at io.confluent.connect.jdbc.sink.DbStructure.createOrAmendIfNecessary(DbStructure.java:67)
        at io.confluent.connect.jdbc.sink.BufferedRecords.add(BufferedRecords.java:122)
        at io.confluent.connect.jdbc.sink.JdbcDbWriter.write(JdbcDbWriter.java:74)
        at io.confluent.connect.jdbc.sink.JdbcSinkTask.put(JdbcSinkTask.java:90)
        at org.apache.kafka.connect.runtime.WorkerSinkTask.deliverMessages(WorkerSinkTask.java:587)
        ... 11 more
```

### 解決

原因是因為 postgresql 沒辦法處理資料型態，所以需要在在 Oracle source connector 指定 "decimal.handling.mode": "double" 或是 "decimal.handling.mode": "string" 來轉換格式。

查看原表發現被讀成 VariableScaleDecimal 的資料欄位 ALERT_FLAG 實際上是 NUMBER 資料型態，對比其他正常被讀取成 int 的欄位，差別在於沒有定義固定精確度和零位小數位數。所以被處理成 decimal。

![](https://imgur.com/g4q5PtT.png)

在加上參數後，可以發現重新佈署的 connector 收進來的資料就改變成指定型態了。

![](https://imgur.com/y0A4akV.png)

不過因為舊的帶有 decimal struct 格式的資料還是存在在 kafka topic，所以如果再重新佈署 postgresql sink connector 的話仍會因為讀到舊的資料而失敗，除非砍掉 topic 重新佈署。

### Reference

- https://github.com/dursunkoc/kafka_connect_sample/tree/main
- https://github.com/pranav1699/debezium-kafka-cdc/blob/main/debezium-jdbc-es/Dockerfile
- https://medium.com/@parasharprasoon.950/how-to-set-up-cdc-with-kafka-debezium-and-postgres-70a907b8ca20
