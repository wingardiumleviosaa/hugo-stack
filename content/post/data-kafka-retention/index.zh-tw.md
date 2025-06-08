---
title: "[Kafka] retention.bytes 和 retention.ms 配置對訊息保留的影響"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2024-03-16 10:43:00
slug: "kafka-retention-setting"
---

在 Apache Kafka 中，當訊息的 `cleanup.policy` 為 `delete` 時配置 `retention.bytes` 和 `retention.ms` 會影響訊息保留的時間跟大小。本文將探討這兩個參數如何共同影響消息的保留，以及如何正確設置它們以滿足特定需求。

<!--more-->

## 問題描述

在 Kafka 主題中，如果設置了以下參數：

- **`retention.ms`**: 15552000000（約 180 天）
- **`retention.bytes`**: 1073741824（1 GB）

期望行為是按照時間保留消息 180 天，但由於 **`retention.bytes`** 的限制，可能導致消息在未達到 180 天前就被刪除。

## 原因分析

Kafka 使用 **`retention.bytes`** 和 **`retention.ms`** 兩個參數來控制消息的保留：

1. **retention.ms**：設定消息保留的最長時間。
2. **retention.bytes**：設定可以在日誌中保留的最大數據量。

當 **`retention.bytes`** 的值設置為一個具體的數字時，即使消息還沒有達到 **`retention.ms`** 指定的時間，一旦存儲的數據量超過 **`retention.bytes`** 指定的限制，最舊的數據將被刪除。

## 解決方案

若要按時間保留消息，而不是基於存儲量，應進行以下設置：

- **設置 `retention.ms`**：根據需要保留消息的時間設置此參數。
- **設置 `retention.bytes` 為 `-1`**：這樣做將移除基於大小的限制，僅按時間保留消息。

例如，要保留消息 180 天，無論大小，配置應該是：

```
retention.ms=15552000000  # 約 180 天
retention.bytes=-1        # 無大小限制
```

## 結論

在配置 Kafka 時，了解 **`retention.bytes`** 和 **`retention.ms`** 如何共同作用非常重要。為避免不必要的數據丟失，當目的是基於時間保留數據時，應將 **`retention.bytes`** 設為 **`-1`**。這樣可以確保數據根據時間而非大小被保留或刪除。
