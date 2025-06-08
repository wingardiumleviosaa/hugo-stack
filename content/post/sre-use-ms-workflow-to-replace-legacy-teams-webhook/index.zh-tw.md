---
title: "Use Microsoft Workflow to Replace Legacy Teams Webhook for Notification"
tags:
  - Observability
categories:
  - SRE
date: 2024-12-08T07:11:25+08:00
slug: sre-use-ms-workflow-to-replace-legacy-teams-webhook
---

## TL; DR

根據微軟的公告，Office Connector 即將要退役，取而代之的是 Workflow。本篇記錄如何使用 Workflow 發送 Prometheus 的告警訊息到 Teams Channel。

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/5d999e12-b952-4855-a6f2-530b3451f4a2/Untitled.png)

## 目標

要能在 Prometheus 觸發告警時，通過 Alertmanager 路由到 Teams 的頻道並透過 Adaptive Card 顯示警報訊息與客製化的集中告警管理平台的超連結按鈕。

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/96c7f1dc-a460-4039-919a-d6796afa111d/Untitled.png)

## STEP1：建立流程

在範本中找 “收到 webhook 要求時發布在頻道中” 的 template，

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/2c837359-654d-4611-9a43-631c534e543d/Untitled.png)

## STEP2：編輯流程

### 2-1 新增資料處理動作編輯輸入的訊息

上一步驟選擇的模板會自動產生輸入跟輸入的動作，首先需要在中間新增一個資料編輯的動作，

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/81416fc1-eeac-443c-b180-b93b3a2fa0cd/Untitled.png)

編輯內容，將以下文字貼到 `輸入` 中。

```yaml
[
  {
    "content":
      {
        "type": "AdaptiveCard",
        "version": "1.2",
        "body":
          [
            {
              "type": "TextBlock",
              "weight": "Bolder",
              "size": "ExtraLarge",
              "text": "@{triggerBody()?['title']}",
            },
            { "type": "TextBlock", "text": "@{triggerBody()?['text']}" },
          ],
        "actions":
          [
            {
              "type": "Action.OpenUrl",
              "title": "Open Alerta",
              "url": "https://alerta.sdsp-stg.com/alerts",
            },
          ],
        "msteams": { "width": "Full" },
      },
  },
]
```

其中單身使用到簡單的 `TextBlock` 渲染訊息，最後加上 `Action.OpenUrl` 的按鈕，可參考[文檔](https://adaptivecards.io/explorer/AdaptiveCard.html)介紹。

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/d6ae6095-890f-4240-8da6-5a7e233b2b21/Untitled.png)

### 2-2 修改輸出的 Teams 連接器設定

將要送往的頻道資訊設定好，並修改 `Adaptive Card` 的來源為上一 `編輯` 步驟的 `輸出`。

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/e9f586bc-98f2-447c-9548-6f3d9988b347/Untitled.png)

## STEP3：驗證

複製 Webhook API，並設定 AlertManager 的 Routes

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/15e0c092-20e6-4ea1-9a12-0f094ed99cef/aac59cc8-31ed-4fd4-b1df-b8854d9b5116/Untitled.png)

使用 Kubernetes 環境中 Prometheus Operator 驗證，故修改 kube-prometheus-stack 的 helm chart values.yaml 檔案

```yaml
alertmanager:
  config:
    global:
      resolve_timeout: 5m
    route:
      group_by: ["namespace"]
      group_wait: 3s
      group_interval: 5m
      repeat_interval: 10m
      receiver: "null"
      routes:
        - receiver: "null"
          matchers:
            - "alertname = Watchdog"
        - receiver: "null"
          matchers:
            - "alertname = InfoInhibitor"
        - receiver: "critical-warn-receiver"
          matchers:
            - "severity =~ critical|warning"
    receivers:
      - name: "null"
      - name: "critical-warn-receiver"
        msteams_configs:
          - webhook_url: "https://prod2-18.southeasta...dEAzKeKGjhWHpdhs6jDZnuZvoLW-DEk"
        webhook_configs:
          - url: "http://172.22.51.210/api/webhooks/prometheus?api-key=O..ee"
            send_resolved: true
```

```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring --version 60.5.0 --values ./values.yaml
```

等待告警發生後就可以在 Channel 中看到卡片了。

## Reference

- https://github.com/prometheus/alertmanager/issues/3920
