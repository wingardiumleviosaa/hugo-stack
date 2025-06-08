---
title: "Apache Paimon Deployment Guide with Simple Demo"
tags:
  - Paimon
  - Ceph
categories:
  - DataEngineering
  - ApacheFlink
date: 2024-06-01T15:06:01+08:00
slug: data-paimon-deploy-guide
---

## 概述

紀錄如何部署 Apache Paimon。

<!--more-->

## Prerequisite

1. **計算引擎設置**：確保 Flink 正在運行。
2. **文件系統準備**：根據存儲偏好選擇 Hadoop 分佈式文件系統(HDFS)或 S3(Ceph/Minio)。

## JAR 文件設置

要將 Apache Paimon 與選擇的文件系統集成，必須將特定的 JAR 文件放置在 Flink 的庫目錄中：

1. **核心功能**：添加`paimon-flink-1.17-0.7.0-incubating.jar`。
2. **Hadoop 部署**：包括`hadoop-mapreduce-client-core-*.jar`和`flink-shaded-hadoop-2-uber-2.8.3-10.0.jar`。
3. **S3 部署**：確保添加`paimon-s3-0.7.0-incubating.jar`。

## Flink 配置

配置 Flink 以適應您的部署後端和檢查點要求：

1. **範例配置**：

   ```yaml
   state.backend.type: rocksdb
   state.checkpoints.dir: hdfs://hadoop1:8020/flink-checkpoints
   state.checkpoints.incremental: true
   ```

## Catalog 設置

Apache Paimon 支持各種目錄，設置因文件系統而異：

| 文件系統     | URI 方案 | 可插拔 | 說明                               |
| ------------ | -------- | ------ | ---------------------------------- |
| 本地文件系統 | file://  | 否     | 內建支持                           |
| HDFS         | hdfs://  | 否     | 內建支持，確保集群在 Hadoop 環境中 |
| 阿里雲 OSS   | oss://   | 是     | -                                  |
| S3           | s3://    | 是     | -                                  |

### Hadoop 目錄

- 利用對 HDFS 的內建支持，創建適用於 Hadoop 環境的目錄。
  ```sql
  CREATE CATALOG nexdata WITH (
      'type' = 'paimon',
      'warehouse' = 'hdfs://hadoop1:8020/paimon/nexdata'
  );
  USE CATALOG nexdata;
  ```

### S3 目錄（Ceph/Minio）

- 配置 S3 兼容存儲的目錄，根據需要調整 Ceph 或 Minio 的設置。
  - **Ceph 範例**：
    ```sql
    CREATE CATALOG `paimon_ceph` WITH (
      'type' = 'paimon',
      'warehouse' = 's3://ula-test/paimon',
      's3.endpoint' = 'http://ceph-rgw.sdsp-stg.com:8080',
      's3.access-key' = '0V2QOGVWCZAAK9DXA07P',
      's3.secret-key' = '60Kwr0A9jjH4A2ZOHgY74ZBUbpxgG2ALsoQ6uC4W',
      'fs.s3a.signing-algorithm' = 'S3SignerType',
      's3.path.style.access' = 'true'
    );

    ```
  - **Minio 範例**：
    ```sql
    CREATE CATALOG `paimon_minio` WITH (
        'type' = 'paimon',
        'warehouse' = 's3://ula/paimon',
        's3.endpoint' = 'http://192.168.31.3:9000',
        's3.access-key' = 'RyUWQB41XSEw7usT',
        's3.secret-key' = 'if58BDcWfJaaP13pxkOcAcGkcQMN0fvm',
        's3.path.style.access' = 'true'
    );

    ```

## 定義數據結構（DDL）

使用數據定義語言在 Apache Paimon 中定義您的數據結構：

1. **MES 生產線數據表**：
   - 根據您的數據建模需求製作表格，容納如 MES 生產線數據表等結構。
     ```sql
     CREATE TABLE IF NOT EXISTS `ods_mes_line` (
         `LINE_CODE` INT,
         `LINE_NAME` STRING,
         `LINE_TYPE` STRING,
         `LINE_DESC` STRING,
         `LINE_ABBREV` STRING,
         `TIME_SECTION_CODE` INT,
         `USABLE_FLAG` INT,
         `UPDATE_EMP` STRING,
         `UPDATE_TIME` BIGINT,
         PRIMARY KEY (`LINE_CODE`) NOT ENFORCED
     );

     ```
2. **來自 Kafka CDC 的 MES 生產線數據表**
   - 創建來自 Kafka 的 MES 生產線數據表：
     ```sql
     CREATE TEMPORARY TABLE `mes_line` (
         `LINE_CODE` INT,
         `LINE_NAME` STRING,
         `LINE_TYPE` STRING,
         `LINE_DESC` STRING,
         `LINE_ABBREV` STRING,
         `TIME_SECTION_CODE` INT,
         `USABLE_FLAG` INT,
         `UPDATE_EMP` STRING,
         `UPDATE_TIME` BIGINT,
         PRIMARY KEY (`LINE_CODE`) NOT ENFORCED
     ) WITH (
     'connector' = 'kafka',
     'topic' = 'MES-INFO.EMESC.TC_WS_LINE_DESC',
     'properties.bootstrap.servers' = 'kafka.sdsp-stg.com:9094',
     'properties.group.id' = 'streamparkTest',
     'scan.startup.mode' = 'earliest-offset',
     'value.debezium-json.schema-include' = 'true',
     'value.format' = 'debezium-json'
     );

     ```

## 查詢數據

{{< notice note >}}
💡 在 Apache Paimon 中執行查詢。根據目錄設置和數據的性質（流式或批量），方法略有不同。
{{< /notice >}}

通過設置適當的運行模式來執行流式查詢：

```sql
SET 'execution.runtime-mode'='streaming';
SELECT * FROM ods_mes_line;
```
