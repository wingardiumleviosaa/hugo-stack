---
title: "Apache Flink and Paimon Quick Start with Docker Compose"
tags:
  - Paimon
  - Flink
categories:
  - DataEngineering
  - ApacheFlink
date: 2024-06-09T10:06:01+08:00
slug: data-flink-paimon-quick-start-with-docker-compose
---

## TL; DR

紀錄使用 docker compose 快速部署 Apache Flink，並使用 Apache Paimon 做為 Source 存放資料。

<!--more-->

## Prepare the necessary JAR files into Flink's library directory

1. Paimon Flink JAR
2. Paimon S3 JAR
3. Hadoop Bundled JAR

## Deploy

```yaml
version: "3"
services:
  jobmanager:
    image: flink:1.17.2-scala_2.12
    ports:
      - "8081:8081"
    command: jobmanager
    user: flink
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
    networks:
      - flink-network
    volumes:
      - ./custom_jars/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar:/opt/flink/lib/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar
      - ./custom_jars/paimon-s3-0.7.0-incubating.jar:/opt/flink/lib/paimon-s3-0.7.0-incubating.jar
      - ./custom_jars/paimon-flink-1.17-0.7.0-incubating.jar:/opt/flink/lib/paimon-flink-1.17-0.7.0-incubating.jar
      - ./custom_jars/flink-sql-avro-confluent-registry-1.17.2.jar:/opt/flink/lib/flink-sql-avro-confluent-registry-1.17.2.jar
      - ./custom_jars/flink-sql-connector-kafka-1.17.2.jar:/opt/flink/lib/flink-sql-connector-kafka-1.17.2.jar
      - ./tmp:/tmp

  taskmanager:
    image: flink:1.17.2-scala_2.12
    depends_on:
      - jobmanager
    command: taskmanager
    user: flink
    scale: 1
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        taskmanager.numberOfTaskSlots: 4
    networks:
      - flink-network
    volumes:
      - ./custom_jars/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar:/opt/flink/lib/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar
      - ./custom_jars/paimon-s3-0.7.0-incubating.jar:/opt/flink/lib/paimon-s3-0.7.0-incubating.jar
      - ./custom_jars/paimon-flink-1.17-0.7.0-incubating.jar:/opt/flink/lib/paimon-flink-1.17-0.7.0-incubating.jar
      - ./custom_jars/flink-sql-avro-confluent-registry-1.17.2.jar:/opt/flink/lib/flink-sql-avro-confluent-registry-1.17.2.jar
      - ./custom_jars/flink-sql-connector-kafka-1.17.2.jar:/opt/flink/lib/flink-sql-connector-kafka-1.17.2.jar
      - ./tmp:/tmp

  sql-client:
    image: flink:1.17.2-scala_2.12
    command: bin/sql-client.sh
    user: flink
    depends_on:
      - jobmanager
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        rest.address: jobmanager
    networks:
      - flink-network
    volumes:
      - ./custom_jars/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar:/opt/flink/lib/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar
      - ./custom_jars/paimon-s3-0.7.0-incubating.jar:/opt/flink/lib/paimon-s3-0.7.0-incubating.jar
      - ./custom_jars/paimon-flink-1.17-0.7.0-incubating.jar:/opt/flink/lib/paimon-flink-1.17-0.7.0-incubating.jar
      - ./custom_jars/flink-sql-avro-confluent-registry-1.17.2.jar:/opt/flink/lib/flink-sql-avro-confluent-registry-1.17.2.jar
      - ./custom_jars/flink-sql-connector-kafka-1.17.2.jar:/opt/flink/lib/flink-sql-connector-kafka-1.17.2.jar
      - ./tmp:/tmp

networks:
  flink-network:
    driver: bridge
```

啟動

```
docker compose up -d
```

## Enter Flink SQL client

```sql
docker compose run --rm sql-client
```
