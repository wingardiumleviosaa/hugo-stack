---
title: "[Kafka] 新增 broker 節點並平衡 topic"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2020-08-24 14:10:00
slug: "kafka-add-broker-and-balance-topic"
---

在 Kafka 集群中增加 broker 非常方便，只需為其分配唯一的 broker ID，指定集群使用的 zookeeper connect 位址，然後在新服務器上啟動 Kafka。但是，**舊有的 Topic 不會將 Partition 均勻分配到新的節點上，進而造成數據不平衡。** 因此需要將一些現有數據遷移到這些新機器上進行 rebalancing。

Kafka 提供了 `kafka-reassign-partitions.sh` 工具來進行手動平均分配：

<!--more-->

```
This tool helps to moves topic partitions between replicas.
Option                                  Description
------                                  -----------
--bootstrap-server <String: Server(s)   the server(s) to use for
  to use for bootstrapping>               bootstrapping. REQUIRED if an
                                          absolute path of the log directory
                                          is specified for any replica in the
                                          reassignment json file
--broker-list <String: brokerlist>      The list of brokers to which the
                                          partitions need to be reassigned in
                                          the form "0,1,2". This is required
                                          if --topics-to-move-json-file is
                                          used to generate reassignment
                                          configuration
--command-config <String: Admin client  Property file containing configs to be
  property file>                          passed to Admin Client.
--disable-rack-aware                    Disable rack aware replica assignment
--execute                               Kick off the reassignment as specified
                                          by the --reassignment-json-file
                                          option.
--generate                              Generate a candidate partition
                                          reassignment configuration. Note
                                          that this only generates a candidate
                                          assignment, it does not execute it.
--help                                  Print usage information.
--reassignment-json-file <String:       The JSON file with the partition
  manual assignment json file path>       reassignment configurationThe format
                                          to use is -
                                        {"partitions":
                                                [{"topic": "foo",
                                                  "partition": 1,
                                                  "replicas": [1,2,3],
                                                  "log_dirs": ["dir1","dir2","dir3"]
                                          }],
                                        "version":1
                                        }
                                        Note that "log_dirs" is optional. When
                                          it is specified, its length must
                                          equal the length of the replicas
                                          list. The value in this list can be
                                          either "any" or the absolution path
                                          of the log directory on the broker.
                                          If absolute log directory path is
                                          specified, the replica will be moved
                                          to the specified log directory on
                                          the broker.
--replica-alter-log-dirs-throttle       The movement of replicas between log
  <Long: replicaAlterLogDirsThrottle>     directories on the same broker will
                                          be throttled to this value
                                          (bytes/sec). Rerunning with this
                                          option, whilst a rebalance is in
                                          progress, will alter the throttle
                                          value. The throttle rate should be
                                          at least 1 KB/s. (default: -1)
--throttle <Long: throttle>             The movement of partitions between
                                          brokers will be throttled to this
                                          value (bytes/sec). Rerunning with
                                          this option, whilst a rebalance is
                                          in progress, will alter the throttle
                                          value. The throttle rate should be
                                          at least 1 KB/s. (default: -1)
--timeout <Long: timeout>               The maximum time in ms allowed to wait
                                          for partition reassignment execution
                                          to be successfully initiated
                                          (default: 10000)
--topics-to-move-json-file <String:     Generate a reassignment configuration
  topics to reassign json file path>      to move the partitions of the
                                          specified topics to the list of
                                          brokers specified by the --broker-
                                          list option. The format to use is -
                                        {"topics":
                                                [{"topic": "foo"},{"topic": "foo1"}],
                                        "version":1
                                        }
--verify                                Verify if the reassignment completed
                                          as specified by the --reassignment-
                                          json-file option. If there is a
                                          throttle engaged for the replicas
                                          specified, and the rebalance has
                                          completed, the throttle will be
                                          removed
--version                               Display Kafka version.
--zookeeper <String: urls>              REQUIRED: The connection string for
                                          the zookeeper connection in the form
                                          host:port. Multiple URLS can be
                                          given to allow fail-over.
```

---

## 新增 kafka broker 到 cluster 中

首先先到集群的 zookeeper 中開啟 zkCli 查看目前集群下有的 kafka broker

![](https://imgur.com/FkFrL6U.png)

準備一台新的節點，其中配置檔的 `id` 要寫不同於群集現有的 id，`zookeeper.connect` 填寫群集使用的 zookeeper。例如：

```
broker.id=3
zookeeper.connect=10.1.5.31:2181,10.1.5.32:2181,10.1.5.33:2181/kafka
```

啟用節點

![](https://imgur.com/7bkElf4.png)

回到 zkCli 查看 kafka broker id，就可以發現節點已經被加入叢集了！

![](https://imgur.com/njWtLVh.png)

---

## partition reassign

原有的 topic `test` 有三個 replication 以及三個 partition，ISR 如下：

![](https://imgur.com/TvoSa8i.png)

在新增 broker 節點後，可以使用 kafka-reassign-partitions.sh 重新分配 partition 的分布。

### 1. 建立 `topics.json` 檔案

```json
{
  "version": 1,
  "topics": [{ "topic": "test1" }, { "topic": "test2" }]
}
```

其中 `topics`是一個 array，可以在其中放置多個 topics。 但是在生產環境中最好一個接一個地進行 reassign，這樣就不會因為一個 topic 重新平衡失敗而影響整個集群。

### 2. 生成分配計畫

```
$ cd /opt/kafka
$ bin/kafka-reassign-partitions.sh --zookeeper 10.1.5.31:2181,10.1.5.32:2181,10.1.5.33:2181/kafka --topics-to-move-json-file topic.json  --broker-list  "0,1,2,3"  --generate
```

- --generate: 根據給與的 Topic 列表和 Broker 列表生成遷移計劃。 generate 並不會真正進行消息遷移，而是將消息遷移計劃計算出來，供 execute 命令使用。

![](https://imgur.com/KHNRdTr.png)

`Current partition replica assignment` 表示當前的消息存儲狀況。 `Proposed partition reassignment configuration`表示遷移後的消息存儲狀況。

### 3. 執行重分配計畫

將上面產出的遷移列表 `Proposed partition reassignment configuration` 存入文件 reassignment.json 中，供 --execute 命令使用。

![](https://imgur.com/8ovsqWl.png)
執行計畫

```
$ bin/kafka-reassign-partitions.sh --zookeeper 10.1.5.31:2181,10.1.5.32:2181,10.1.5.33:2181/kafka --reassignment-json-file reassignment.json --execute
```

- --execute: 根據 topic 遷移計劃進行遷移。
- --throttle 為了避免遷移過程中影響 kafka 正常使用，可以加入這個參數限制遷移的流量，單位是 byte。

## 4. 驗證進度

```
$ bin/kafka-reassign-partitions.sh --zookeeper 10.1.5.31:2181,10.1.5.32:2181,10.1.5.33:2181/kafka --reassignment-json-file reassignment.json --verify
```

- --verify: 檢查消息是否已經遷移完成。

![](https://imgur.com/n4zrCwN.png)

## 5. 查看 topic 副本狀況

```
$ bin/kafka-topics.sh --describe --bootstrap-server 10.1.5.31:9092 --topic test
```

![](https://imgur.com/5mY3pG8.png)

## preferred replica election

關於優先副本的選舉可以保持預設值讓節點間自動平衡，在 broker 啟動時，因預設 `auto.leader.rebalance.enable` 的默認值為 true，故當 broker 在故障轉移操作時會啟動一個定時任務，每隔 `${leader. imbalance.check.interval.seconds}` 秒 (默認是 5 分鐘) 觸發一次分區分配均衡操作，而只有在代理的不均衡的百分比達到 `${leader.imbalance.per.broker.percentage}` (default 10，即不均衡比例達到 10%) 以上時才會真正執行分區重新分配操作。
