---
title: >-
  使用 alpine base image 在指定時間定期執行 CronJob
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-10-23 19:08:00
slug: cronjob-for-last-day-of-the-month-using-kubernetes
---
## TL;DR
本篇文章記錄如何在 aplpine base 的 container 中於每月的最後一個禮拜日執行指定任務。

<!--more-->

## Soluiton
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
  namespace: test
spec:
  schedule: "0 8 * * 0"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: alpine:3.16
            resources:
              limits:
                cpu: "2"
                memory: 4000Mi
            imagePullPolicy: Always
            command:
            - /bin/sh
            - -c
            - '[ $(date +%m) -ne $(date -d "@$(($(date +%s) + 604800))" +%m) ] && echo Hello from the Kubernetes cluster'
          restartPolicy: OnFailure
```

## 坑
原本前面判斷的語法是寫在 ubuntu 下 run 得好好的 `[ $(date +%m) -ne $(date -d +7days +%m) ]`，但是殊不知搬到 alpine container 後會報錯 `date: invalid date '+7days'`。所以改寫成 apline 環境中認得的 -d 格式！

## Reference
- https://stackoverflow.com/questions/71651529/schedule-cronjob-for-last-day-of-the-month-using-kubernetes
- https://unix.stackexchange.com/a/522622