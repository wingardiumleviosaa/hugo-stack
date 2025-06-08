---
title: "[Zookeeper] fsync & EndOfStreamException 導致 zookeeper shutdown"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["ZooKeeper"]
slug: "zookeeper-endofstreamexception"
date: 2020-07-31 14:50:00
---

## 問題

Zookeeper 及 Kafka 在參數保持預設的狀況下，做壓力測試時 Zookeeper 會發生以下錯誤訊息：

<!--more-->

1. fsync-ing the write ahead log in SyncThread

```
2020-07-24 08:24:15,665 [myid:1] - WARN [SyncThred:2:FileTxnLog@408] - fsync-ing the write ahead log in SyncThread:3 took 8154ms which will adversely effect operation latency.File size is 67108880 bytes. See the ZooKeeper troubleshooting guide
```

2. EndOfStreamException: Unable to read additional data from client, it probably closed the socket

```
2020-07-24 08:26:38,929 [myid:1] - WARN  [NIOWorkerThread-2:NIOServerCnxn@364] - Unexpected exception
EndOfStreamException: Unable to read additional data from client, it probably closed the socket: address = /10.1.5.33:51856, session = 0x200150baa0a0004
org.apache.zookeeper.server.NIOServerCnxn.handleFailedRead(NIOServerCnxn.java:163)
org.apache.zookeeper.server.NIOServerCnxn.doIO(NIOServerCnxn.java:326)
org.apache.zookeeper.server.NIOServerCnxnFactory$IOWorkRequest.doWork(NIOServerCnxnFactory.java:522)
org.apache.zookeeper.server.WorkerService$ScheduledWorkRequest.run(WorkerService.java:154)
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
 at java.lang.Thread.run(Thread.java:748)
2020-07-24 08:26:48,440 [myid:1] - info  [NIOWorkerThread-3:Learner@137] - Revalidating client: 0x200150baa0a0004
```

出現後通常會伴隨 Zookeeper 集群 shutdown，雖然過一段時間後即群會自動回復，但是整體性能會大幅下降。

## 配置

先說明一下機器環境及壓力測試條件：

### 環境

CPU 24 cores
RAM 256 GB
網卡 10 GB
共三台 server 各跑一個 Zookeeper 以及一個 kafka container。

### 測試

模擬 10 個 client，每個 clinet 丟 5000 萬筆訊息，每筆訊息大小為 1K，Kafka topic 的分區數及副本數都設為 3。

## 解決方法

1. session timeout
   問題的原因在於，配置的超時時間太短，Zookeeper 沒有讀完 Consumer (這裡指 Kafka) 的數據，連接就被 Consumer 斷開了。
   所以在 Kafka 的 server.properties 文件中將針對 Zookeeper 的超時連接屬性的值調大一點，例如 :

```
# server.properties
zookeeper.session.timeout.ms=500000
```

2. fysnc
   發生原因是因為 ZooKeeper 將數據 write ahead log(WAL) 寫入磁碟的速度過慢，導致 ZooKeeper 節點間 heartbeat (follower 需與同步 leader) 超時。
   根本解決的方式是掛載新的硬碟到 Zookeeper 節點機器上，並建議將應用如 Kafka 與 Zookeeper 的機器分開，提高磁碟 IO 性能。
   折衷處理方式則是提高 Zookeeper/config/zoo.cfg 中**tickTime** 以及 **fsync.warningthresholdms** 參數。

```
# zoo.cfg
tickTime=4000
fsync.warningthresholdms=10000
```

## 補充

- Zookeeper 儲存機制
  ZooKeeper 主要是在記憶體中維護數據，但每個改變都會被寫入一個在存儲介質上的持久 WAL（Write Ahead Log）。當一個服務故障時，它能夠通過回放 WAL 恢復之前的狀態。為了防止 WAL 無限制的增長，ZooKeeper 服務會定期的將記憶體狀態快照保存到存儲介質。這些快照能夠直接加載到內存中，所有在這個快照之前的 WAL 條目都可以被安全的丟棄。
- fsync.warningthresholdms 參數
  用於配置 Zookeeper 進行 WAL fsync 操作時消耗時間的報警閾值。一旦 fsync 操作消耗的時間大於該參數指定的值，就在日誌中打印出報警日誌。

## Reference

[1] https://zookeeper.apache.org/doc/r3.4.10/zookeeperAdmin.html  
[2] https://blog.csdn.net/levy_cui/article/details/52242715?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param  
[3] https://v1-16.docs.kubernetes.io/zh/docs/tutorials/stateful-application/zookeeper/
