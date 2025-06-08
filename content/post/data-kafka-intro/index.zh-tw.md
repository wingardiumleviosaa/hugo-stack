---
title: "[Kafka] Kafka at a Glance"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2020-06-09 10:43:00
slug: "kafka-at-glance"
---

Kafka 是由 LinkedIn 開發而後釋出給 Apache 的一個開源 streaming message queue (消息佇列)，由 Scala 和 Java 編寫，目前是 Hadoop ecosystem 中的一員。

<!--more-->

![](https://imgur.com/03l9i3Y.png)

## 特色

以下簡單列出 Kafka 幾點特色：

- 基於**發布與訂閱** (Publish & Subscribe)
- 架構中有三個角色，`broker`, `producer` 以及 `consumer`
- Broker 為訊息系統本身，負責接收並儲存資料
- Producer 負責發布消息到 broker
- Consumer 負責從 borker 訂閱消息
- Producer 以及 Consumer 於 broker 來說是 client
- 多個 brokers 可以組成一個 cluser，叢集具有**負載平衡**以及**方便擴展**而不會影響運作的特性

![](https://imgur.com/j6HnRWD.jpg)

- Kafka 在儲存、發布以及訂閱訊息時是透過 `Topic` 進行歸類
- 每一個 Topic 可包含一個或多個 `Partition`，partition 是資料實際儲存的地方，增加 partition 可以**提高吞吐量**
- 可以透過設定副本 replication-factor 達到**高可靠性**
- 資料儲存在硬碟，可指定存放的時間或是大小，達到**持久性**

## Page Cache

Linux 內核會將系統中所有的空閒記憶體全部當做 page cache 來用，而 page cache 中的所有 page 數據將一直保存在 page cache 中直到 CPU 根據特定的算法替換掉它們中的某些 page。
當 produce 資料到 Kafka 時，會先將數據寫入 Page Cache 中，並將該 page 上有 dirty 標誌。當 consumer 向 Kafka 讀取資料時，會先在 Page Cache 中查找內容，如果有就直接返回，沒有的話就會從 disk 讀取文件再寫回 Page Cache。
因此只要生產者與消費者的速度相差不大，消費者會直接讀取之前生產者寫入 Page Cache 的數據，大家在記憶體內裡完成接力，沒有磁碟訪問。
比起在記憶體中維護一份消息數據的傳統做法，這既不會重複浪費一倍的記憶體，Page Cache 又不需要 GC，而且即使 Kafka 重啟了，Page Cache 還依然在。
