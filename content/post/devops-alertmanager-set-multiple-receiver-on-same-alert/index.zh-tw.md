---
title: "Alertmanager 將同一個告警送到多個 Receivers"
tags:
  - Observability
categories:
  - SRE
date: 2024-06-10T12:11:53+08:00
slug: devops-alertmanager-set-multiple-receiver-on-same-alert
---

## TL; DR

本篇記錄怎麼將同一個告警路由到兩個不同的接收器中。

<!--more-->

## Solution

```yaml
global:
  resolve_timeout: 5m
route:
  receiver: "dummy"
  group_by: ["alertname"]
  routes:
    - receiver: "teams"
      continue: true
    - receiver: "alerta"
receivers:
  - name: "dummy"
  - name: "teams"
    msteams_configs:
      - webhook_url: "請輸入 teams 的 webhook url"
  - name: "alerta"
    webhook_configs:
      - url: "http://172.20.37.21:8099/api/webhooks/prometheus?api-key=ek1emONw-2IS6yc31nziqiawcpCpmnAiLXuMqgbq"
        send_resolved: true

templates:
  - "/etc/alertmanager/template/*.tmpl"
```

## 設定思路

- **receiver: "dummy"**: 根路由的默認接收者，如果告警沒有匹配到任何子路由，則會使用根路由指定的接收者來處理告警。根路由必須指定一個接收者，因此設置 `dummy` 接收者來滿足這個要求，這個接收者實際上不執行任何動作。

  所以如果使用下面配置是無法達成目的的，因為匹配到子路由後，就不會再往根路由的接收器發送了。

  ```yaml
  global:
    resolve_timeout: 5m
  route:
    receiver: "teams"
    group_by: ["alertname"]
    repeat_interval: 3s
    group_wait: 3s
    group_interval: 3s
    routes:
      - receiver: "alerta"
  receivers:
    - name: "teams"
      msteams_configs:
        - webhook_url: "請輸入 teams 的 webhook url"
    - name: "alerta"
      webhook_configs:
        - url: "http://172.20.37.21:8099/api/webhooks/prometheus?api-key=ek1emONw-2IS6yc31nziqiawcpCpmnAiLXuMqgbq"
          send_resolved: true

  templates:
    - "/etc/alertmanager/template/*.tmpl"
  ```

  `continue: true` 無法設定在根路由當中，會報 `cannot have continue in root route` 的錯誤。

  ![](./err.png)

- **routes**: 定義了兩個子路由：每個告警都會從設定檔中頂 root route 進入路由樹，root route 會匹配所有警告 (即不會有任何的 match 匹配設定)，routes 中的每一個 route 都可以定義 receiver 及匹配規則。預設情況下，警告進入到 root route 後會遍歷所有的子節點，直到找到匹配 route 後停止 (意即 continue 預設值為 false)，如果將 continue 設為 true，警報則會繼續進行後續子節點的比對。
  - 第一個子路由將告警發送到 `teams` 接收者，並且 `continue: true` 設定確保這些告警會繼續向下匹配下一個子路由。
  - 第二個子路由將告警發送到 `alerta` 接收者。
