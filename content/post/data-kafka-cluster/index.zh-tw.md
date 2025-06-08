---
title: "[Kafka] 建立 Kafka Cluster (run as daemon)"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
slug: "kafka-cluster"
date: 2020-06-10 20:52:00
---

```bash
cd /opt
wget "http://ftp.tc.edu.tw/pub/Apache/kafka/2.4.1/kafka_2.12-2.4.1.tgz"
sudo tar zxvf kafka_2.12-2.4.1.tgz
sudo mv kafka_2.12-2.4.1 kafka
```

<!--more-->

建立儲存 log 的資料夾，並指定權限

```bash
sudo mkdir /var/lib/kafka
sudo chown -R $(USER id -u):$(USER id -g) /opt/kafka
sudo chown -R $(USER id -u):$(USER id -g) /var/lib/kafka
```

編輯 server.properties 設定檔

```bash
sudo vi /opt/kafka/config/server.properties
```

修改以下參數

```toml
#指定 cluster 中的 broker 的唯一 id
broker.id=0

#指定監聽 ip & 阜號
listeners=INSIDE://[private ip]:9092,OUTSIDE://[private ip]:9094
advertised.listeners= INSIDE://[private ip],OUTSIDE://[public ip]:9094
listener.security.protocol.map= INSIDE:PLAINTEXT,OUTSIDE:PLAINTEXT
inter.broker.listener.name= INSIDE

#指定存放log的路徑
log.dirs=/var/lib/kafka

#指定預設的分區數
num.partitions=3

#允許使用者刪除 topic
delete.topic.enable = true

#加入zookeeper 的位置
zookeeper.connect=10.0.0.4:2181,10.0.0.5:2181,10.0.0.6:2181
```

將 KAFKA 設為服務

```bash
sudo vi /etc/systemd/system/kafka.service
```

加入以下文字

```toml
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=root
ExecStart=/bin/sh -c "/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties > /opt/kafka/kafka.log 2>&1"
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```

啟動 kafka

```=10
sudo systemctl start kafka
sudo systemctl enable kafka
```
