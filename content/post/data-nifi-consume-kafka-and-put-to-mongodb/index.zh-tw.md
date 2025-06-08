---
title: "[nifi] Consume Kafka Topic and Put to MongoDB"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka", "ETL"]
date: 2021-09-03 15:13:00
slug: nifi-consume-kafka
---

這個範例將示範使用 Apache NiFi 消費指定的 kafka topic，並寫進指定的 MongoDB。

<!--more-->

![](https://imgur.com/wOVPXlu.png)

kafka 的資料源為 JSON 結構的 string，如下：

```
"{"Time":1630652601.119,"SMT_B_3_machine_name":null,"SMT_B_3_SN":null,"SMT_B_3_program_number":null,"SMT_B_3_WO":null,"SMT_B_3_whole_OK":null,"SMT_B_3_whole_NG":null,"SMT_B_3_whole_reOK":null,"SMT_B_3_whole_yieldRate":null,"SMT_B_3_board_OK":null,"SMT_B_3_board_NG":null,"SMT_B_3_board_reOK":null,"SMT_B_3_board_yieldRate":null,"SMT_B_3_component_OK":null,"SMT_B_3_component_NG":null,"SMT_B_3_component_reOK":null,"SMT_B_3_component_yieldRate":null,"SMT_B_3_tin_OK":null,"SMT_B_3_tin_NG":null,"SMT_B_3_tin_reOK":null,"SMT_B_3_tin_yieldRate":null}"
```

## 1. ConsumeKafka_2_6

![](https://imgur.com/IfSEnQm.png)

![](https://imgur.com/K1tWR8f.png)

## 2. PutMongo

![](https://imgur.com/zqApWOn.png)

## 3. 建立關係

### a. ConsumeKafka —> PutMongo

![](https://imgur.com/wOVPXlu.png)

### b. 建立完成後可以發現最後一個 PutMongo 的 processor 有報錯

![](https://imgur.com/ROEzhXl.png)

### c. 進入 PutMongo processor 的 `SETTINGS` 將 `Automatically Terminate Relationships` 的關係打開

![](https://imgur.com/R2hAeN5.png)

{{< notice info >}}
`Automatically Terminate Relationships` 指的是數據流路由到這個 Processor 後，特定狀態下會被刪除，一般在 Endpoint Processor 配置，因為數據流不需要再被繼續路由了。
{{< /notice >}}

## 4. 啟動流程

都完成後可以看到 flow 的原件都已經 Ready，將 Processor 依次啟動。

![](https://imgur.com/wOVPXlu.png)

啟動後可以看到數據開始收送，

![](https://imgur.com/oPdDASw.png)

另外通過 NiFi Data Provenance 可以看到數據流的狀態

![](https://imgur.com/UJHbHNR.png)

## Reference

- https://anyisalin.github.io/2019/01/03/nifi-demo/
- https://cloud.tencent.com/developer/article/1416651
