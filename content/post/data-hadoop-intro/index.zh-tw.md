---
title: "[Hadoop] HDFS、MapReduce、Yarn 介紹"
categories: ["DataEngineering"]
tags: ["Hadoop"]
date: 2020-08-12 14:32:00
slug: "hadoop-introduction"
---

Hadoop 是一個能夠儲存並管理大量資料的分散式大數據處理平臺，其包含三大模組：

- HDFS
- MapReduce
- Yarn
<!--more-->

## HDFS

HDFS 為 Hadoop Distributed File System 的縮寫，分散式檔案系統。由 NameNode 與 DataNode 組成。

### NameNode & DataNode

NameNode：儲存檔案的 block 清單，稱之為 metadata。
DataNode：負責儲存實體檔案的 block。

### HDFS Architecture

儲存檔案到 HDFS 前，檔案會被切成數等分小區塊 (block)，並且會將同一個 block 複製成數等分(replication, 預設值是 3 份) 再分散儲存到各個 DataNode，且相同的 block 不會同時存在於同一個 DataNode 上。
同時由 NameNode 管理記載 block 的儲存位置，當某個 Hadoop client 需要讀取這個檔案時，會先跟 NameNode 發出請求，NameNode 會根據這份清單回覆檔案的 block 位於哪幾台 DataNode，Hadoop client 再根據這份清單將各個 block 讀取出來，還原成一個完整個檔案。

![](https://imgur.com/R3nEcsJ.png)

### 特性

擴充性：Hadoop 可以通過增加節點輕易的擴展儲存能力或處理效能。
可靠性：由 block 的複本機制達成，當一個 Data Node 失效、毀損造成資料遺失，可以從其他台 Data Node 可以取得該檔案的複本資料。

---

## MapReduce

MapReduce 是一種程式模型，用於大數據的並行運算。
HDFS 處理好的檔案資料，搭配 Map & Reduce 函數，將資料片段傳送到計算節點（Mapping），由各個節點計算處理之後再做整合（Reducimg），達到分散式計算。

### 模型流程

![](https://imgur.com/6cmcxE2.png)

1. fork：將要執行的 MapReduce 程式複製到 Master 與每一個 Worker 機器中。Master 與 Worker 在 Hadoop 中即為 JobTracker(NameNode) 與 TaskTracker(DataNode)。
2. assign map：Master 決定 Map 程式與 Reduce 程式，分別由哪些 Worker 機器執行。
3. read：將所有的資料區塊，分配到執行 Map 程式的 Worker 機器中進行 Map。在 Hadoop 中 input files 會存放在 HDFS 上，要執行 Map 程式的 Worker node，就會照著 Master 分配的資料 link，到 HDFS 上去找到他要執行的資料區塊
4. local write：將 Map 後的結果存入 Worker 機器的本地磁碟。
5. remote read：執行 Reduce 程式的 Worker 機器，遠端讀取每一份 Map 結果，進行彙整與排序，同時執行 Reduce 程式。
6. write：將使用者需要的運算結果輸出

由於 MapReduce 所有運算的過程都會讀寫檔案，運算效能相較之下就比較慢。運算的功能慢慢的被 Apache Spark 所取代。

---

## Yarn

Yarn 是一個資源管理系統，用來管理分散式運算應用程式所使用的資源。所以在 Hadoop 平台上執行 MapReduce 的應用程式，就需藉由 Yarn 監控和分配資源來確保工作正常運作。由一個 ResourceManager，與至少一台的 NodeManager 組成，數量預設會與 DataNode 相同。

### ResourceManagrt & NodeManager & ApplicationMaster

- ResourceManager(RM)：用來管理與裁決 Hadoop 叢集內資源的使用，擁有絕對的控制權和對資源的分配權。
- NodeManager(NM)：負責監控 Hadoop 叢集內每台機器的資源使用情況，例如 memory, cpu, disk, network 等，並且將資訊回報給 ResourceManager。
- ApplicationMaster(AM)：負責每一個 application 的調度和協調。

### Yarn Architecture

![](https://imgur.com/ihpVFgK.png)

YARN 的具體工作流程為：

1. client 提交 application
2. RM 為該應用分配 Container 與對應 NM 通信，要求 NM 在 Container 中啟動 AM
3. AM 啟動後向 RM 註冊
4. AM 通過輪詢向 RM 申請領用 Container 資源
5. AM 申請到資源後與 NM 通信，要求啟動任務
6. 運行中的程序會向 AM 通過協議通信同步運行狀態與進度
7. 在應用執行期間，client 與 AM 通信同步任務狀態訊息
8. 應用運行結束後，AM 向 RM 註銷並關閉自己， Container 資源釋放

## Reference

- [HdfsDesign](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
- [Hadoop Ecosystem 之 Hadoop 介紹](https://ithelp.ithome.com.tw/articles/10190756)
- [mapreduce](https://chenhh.gitbooks.io/parallel_processing/content/apache_spark/mapreduce.html)
- [YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)
