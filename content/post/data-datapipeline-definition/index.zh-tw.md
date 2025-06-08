---
title: Data Pipeline 意義、要解決什麼、應具備的功能
categories: ["DataEngineering"]
date: 2021-10-07 15:32:00
slug: what-is-data-pipeline
---

## 意義

- 數據管道（Data Pipeline）是一種允許數據通過數據分析過程<span style="background-color:#FCF3CF">從一個位置高效流向另一個位置的軟體，是處理資料流的系統。</span>這就好比一條傳送帶，它能高效、準確地將數據傳送到流程的每一步。
<!--more-->
- <span style="background-color:#FCF3CF">數據管道是提取、轉換、合併、驗證、進一步分析數據和數據可視化的過程自動化。</span>通過消除錯誤並避免瓶頸和延遲，數據管道可提供端到端效率。
- 數據管道將所有數據視為流式數據，因此它們考慮了靈活的架構。無論數據來自靜態源還是實時源，<span style="background-color:#FCF3CF">數據管道都可以將數據流分割成更小的片段，以便並行處理，從而提升了計算能力。</span>
- ETL 是 Data Pipeline 的一部分，一條 Data Pipeline 可以是好幾段 ETL 的組合。 ETL 通常指資料的取、用、存這三個動作而已；而 Data Pipeline 除了 ETL 之外，也可包含執行 ETL 的系統。

## 要解決什麼

傳統使用 ETL tools 可能會遇到的痛點：

**問題 1 ：**

使用的 RDS 類型多種多樣，有 ORALCE、SQL SERVER、MYSQL，甚至有 MONGODB。現在要進行大數據分析，如何整合這些數據庫的數據，到一個大數據平台進行數據分析？如何解決多個數據源數據獲取不及時造成數據獲取延遲？

**問題 2：**

數據設計之初，沒有考慮 ETL 數據抽取的問題，所以數據沒有時間字段，如何在上百 G 的數據中抽取增量數據？

**問題 3：**

面對業務部門多種需求，要求在業務獲得數據的 1 個小時內，將更新的業務數據傳遞到數據部門進行處理，並獲得 DATAVIEW。

**問題 4：**

數據分析人員有的精通 T-SQL、有的擅長 PL/SQL，還有的只會 JAVA ，你如何滿足這樣多種多樣的數據目的地需求？

**問題 5：**

目前由於數據庫更新，將 ORACLE 數據庫替代，使用 PostgresQL 來代替 ORACLE。目前需要進行灰度發布(金絲雀發布)，ORACLE 和 POSTGRESQL 數據之間進行實時同步，當程序跑通，上線兩個禮拜後沒有問題，將 ORACLE 清除，如何做到？

**問題 6：**

數據流是否能及時且一致的傳導到各種目的地，以進行分佈式的運算，同時數據必須在管道進行加工處理且須保證事件只能被處理一次，另外還要保留 raw data 對計算的數據進行驗證。也就是單點多傳、數據清洗、數據整理的要求，數據獲取不准確、數據提供的格式不對、數據提取對系統的負擔...，都可能是箇中問題。

## 應具備的功能

1. 實時的數據流，將業務數據像水一樣的方式，通過水管順暢的流向各個目的端。
2. 支持多種數據庫之間、數據庫與大數據產品之間的任意往來
3. 方便快捷部署，能不在數據源端做任何安裝的數據獲取軟件
4. 可參數化的數據管道，例如可指定提取、轉換和加載數據的開始日期。
5. 針對不同的業務場景使用對應優化的數據結構。寫入場景要用到寫入優化的數據結構，而讀取場景要用到讀取優化的數據結構。
6. 從數據庫底層而不是去用 SQL 來去提取數據，因為在數據生產的時候做處理，性能一定遠遠好過在用戶讀取的時候做處理。 ORACLE REDO、ARCHIVE、SQL SERVER CDC、MYSQL BINLOG、 POSTGRESQL WAL、MONGODB OPLOG 等就是由底層提取數據的方法。
7. 存儲數據的中間狀態，以便可以重試上一次失敗的任務。在工作流中實現結果緩存的方法有很多，例如將中間數據存儲在 Redis 或某些臨時暫存區表中。如果使用諸如 [Prefect](https://docs.prefect.io/core/concepts/results.html#choose-a-result-type) 之類的工作流工具，可以將任務結果以 pickle、JSON 或其他序列化形式，緩存到存儲系統中。
8. 確保足夠小的單一組件大小，如果數據管道是一個工作流而不是一個腳本，則可以直接查看管道中的哪個任務失敗，然後只專注於修復單個損壞的任務即可。工作流中的組是小型且相互獨立的任務集合，這些任務按照特定順序且在特定時間運行，任務之間存在依存關係。
9. 數據清洗，能在數據交換的過程中，還能做點數據的小變動，將不必要的數據，截止在數據的源端的工具。
10. 元數據管理，用來管理整個用戶要用到的所有數據的數據的定義。於這個數據字典，實現一些可複用的數據邏輯。確保在各個系統裡都可以得到清晰一致的數據定義，減少溝通成本。
11. 監控和通知，在數據管道出現故障時發送通知。有人使用 Pagerduty、Slack、Teams 或電子郵件。不管選擇什麼工具，建立一個便於檢查並糾正錯誤的系統至關重要。
12. 統一的可視化管理頁面，提供平台級別的數據管理功能，包括產品權限、數據時效管理和安全管控等方面功能，為數據工程師、運維人員提供直觀的數據任務地圖，隨時可以洞悉數據的最新動態，極大提升運維工作效率和效益。
13. 日誌、日誌、日誌！ last but not least，有了詳盡的日誌，才可容易找出問題在哪。

## Reference

- [https://medium.com/bryanyang0528/data-data-pipeline-101-一-22654343e028](https://medium.com/bryanyang0528/data-data-pipeline-101-%E4%B8%80-22654343e028)
- [https://baijiahao.baidu.com/s?id=1678064530655201694&wfr=spider&for=pc](https://baijiahao.baidu.com/s?id=1678064530655201694&wfr=spider&for=pc)
- [https://read01.com/AzeKMAL.html](https://read01.com/AzeKMAL.html)
- [https://blog.csdn.net/YMPzUELX3AIAp7Q/article/details/103017422](https://blog.csdn.net/YMPzUELX3AIAp7Q/article/details/103017422)
- [https://blog.csdn.net/YMPzUELX3AIAp7Q/article/details/103017431](https://blog.csdn.net/YMPzUELX3AIAp7Q/article/details/103017431)
- [https://zhuanlan.zhihu.com/p/72510621](https://zhuanlan.zhihu.com/p/72510621)
- [https://zhuanlan.zhihu.com/p/337003493](https://zhuanlan.zhihu.com/p/337003493)
- [https://towardsdatascience.com/15-essential-steps-to-build-reliable-data-pipelines-58847cb5d92f](https://towardsdatascience.com/15-essential-steps-to-build-reliable-data-pipelines-58847cb5d92f)
