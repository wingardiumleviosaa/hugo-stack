---
title: 使用 docker compose 安裝 kafka
tags:
  - Kafka
categories:
  - DataEngineering
  - ApacheKafka
date: 2022-03-09 21:54:12
slug: install-kafka-by-docker-compose
---

## 安裝單節點 Kafka

### 1. 準備 docker-compose.yaml 檔案

<!--more-->

```yaml
version: "3"
services:
  zookeeper:
    restart: always
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181
    volumes:
      - ./zoo/data:/var/lib/zookeeper/data
      - ./zoo/log:/var/lib/zookeeper/log

  kafka:
    restart: always
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    volumes:
      - ./kafka/data:/var/lib/kafka/data
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://10.13.1.100:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

- 在使用 docker 或是公有雲部屬 kafka 時，需要用到 `KAFKA_ADVERTISED_LISTENER` 參數。其中第一個值是真正建立 kafka broker 用的，第二個數值是用於對外發布的服務端口。
- `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` 用來配置 `__consumer_offsets` 副本數。

### 2. 啟動 container

```
docker-compose up -d
Creating network "kafka_default" with the default driver
Creating kafka_zookeeper_1 ... done
Creating kafka_kafka_1     ... done
```

### 3. 查看服務

```
docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED          STATUS          PORTS                                                             NAMES
9afa79af69e7   confluentinc/cp-kafka:latest       "/etc/confluent/dock…"   39 minutes ago   Up 39 minutes   9092/tcp, 0.0.0.0:29092->29092/tcp, :::29092->29092/tcp           kafka_kafka_1
2445639c8af9   confluentinc/cp-zookeeper:latest   "/etc/confluent/dock…"   39 minutes ago   Up 39 minutes   2888/tcp, 3888/tcp, 0.0.0.0:22181->2181/tcp, :::22181->2181/tcp   kafka_zookeeper_1

```

### 4. 檢查運作

使用下載 kafka 套件提供的 script 測試

- list topic，若無錯誤訊息則代表建立並連線 kafka broker 成功。

```
[root@ula bin]# ./kafka-topics.sh --list --bootstrap-server 10.13.1.100:29092

[root@ula bin]#
```

- produce message

```
[root@ula bin]# ./kafka-console-producer.sh --bootstrap-server 10.13.1.100:29092 --topic test
>life
>is
>what
>you
>make
>it
>!
>^C[root@ula bin]#
```

- consume message

```
[root@ula bin]#
[root@ula bin]# ./kafka-console-consumer.sh --bootstrap-server 10.13.1.100:29092 --topic test --from-beginning
life
is
what
you
make
it
!
^CProcessed a total of 7 messages
```

## 安裝 Kafka Cluster

```yaml
---
version: "3"
services:
  zookeeper-1:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181
  zookeeper-2:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 32181:2181
  kafka-1:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
      - zookeeper-2
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092,PLAINTEXT_HOST://10.13.1.100:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 2
  kafka-2:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
      - zookeeper-2
    ports:
      - 39092:39092
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-2:9092,PLAINTEXT_HOST://10.13.1.100:39092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 2
```
