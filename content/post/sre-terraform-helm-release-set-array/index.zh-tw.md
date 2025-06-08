---
title: "[Terraform] helm_release Pass Array/List to Set's Value"
tags:
  - Terraform
categories:
  - SRE
date: 2022-05-19 08:58:00
slug: terraform-helm-release-pass-array
---

在 terraform 使用 helm_release，在傳入 set 參數時遇到有 chart 指定要 array 型態的值，但使用 `[]` 傳入 string array 卻會報無法 iterate array。在這邊紀錄一下解法。

<!--more-->

## metrics-server helm chart 描述

這次使用 Terraform 實做安裝的 helm chart 是 metrics-server，[官網](https://artifacthub.io/packages/helm/metrics-server/metrics-server#configuration)提到可以使用 args 傳入啟動 server 的額外參數，啟動參數 `--kubelet-insecure-tls` 目的是防止 metrics server 訪問 kubelet 採集指標時報證書錯誤 `x509: certificate signed by unknown authority` 的問題。

## 原本錯誤的範例

```terraform
resource "helm_release" "metrics-server" {
  count = var.package.metrics == true ? 1 : 0

  name      = "metrics-server"
  namespace = "kube-system"
  chart     = "metrics-server/metrics-server"
  set {
    name  = "args"
    value = "[\"--kubelet-insecure-tls\"]"
  }
}
```

會丟以下錯誤

```bash
Error: template: metrics-server/templates/deployment.yaml:59:27: executing "metrics-server/templates/deployment.yaml" at <.Values.args>: range can't iterate over ["--kubelet-insecure-tls"]
```

## 正確寫法

將 `[]` 括號使用 `{}` 取代，且陣列項目不需要加上 `""`。

```terraform
resource "helm_release" "metrics-server" {
  count = var.package.metrics == true ? 1 : 0

  name      = "metrics-server"
  namespace = "kube-system"
  chart     = "metrics-server/metrics-server"
  set {
    name  = "args"
    value = "{--kubelet-insecure-tls}"
  }
}
```
