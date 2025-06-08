---
title: GitLab CICD 介紹
tags:
  - GitLab
  - CI/CD
categories:
  - DevOps
date: 2022-04-10 16:34:00
slug: gitlab-cicd-intro
---

## CI/CD

![](https://imgur.com/g4KAXij.png)

<!--more-->

## Continuous Integration 持續整合

![](https://imgur.com/rlvl3CZ.png)

指在源代碼變更後的自動檢測(Lint)、構建(Build)和進行單元測試、集成測試(Test)的過程。目標是針對每個變動，能持續性的進行驗證並整合，以確保程式的品質。

## Continuous Delivery 持續交付

![](https://imgur.com/1CAGTMQ.png)

指頻繁地將軟體的新版本，交付給 QA 或使用者，以供評審。如果評審通過，程式碼就進入生產階段。持續交付通常包含了一個手動的步驟，用來讓開發人員確認和部署到生產環境。

## Continuous Deployment 持續佈署

![](https://imgur.com/wd8EhpU.png)

持續部署是指當交付的程式碼通過評審之後，自動部署到生產環境中。持續部署不再需要開發人員手動部署成品到生產環境，只要程式碼提交的 PR 通過了，剩下的整個過程都會自動完成。

## Gitlab CI/CD

![](https://imgur.com/vLgm5hv.png)

### Pipeline

代表一次的完整的自動化任務，每次提交都會觸發一次。根據 GitLab 所述

> pipelines are the top-level component of continuous integration, delivery, and deployment
> pipeline 使用存在 GitLab 項目的根目錄的 yaml 文件 (.gitlab-ci.yml) 定義。

### Stage

代表構建階段，一條 Pipeline 中可定義多個 Stages

- 所有 Stages 會順序執行，即當一個 Stage 完成後，下一個 Stage 才會開始
- 只有當所有 Stages 完成後，該構建任務才會成功
- 如果任何一個 Stage 失敗，那麼後面的 Stages 不會執行，該構建任務失敗

### Job

代表構建工作，是 GitLab CI 中可以獨立控制並運行的最小單位，一個 Stage 中可以有多的 Jobs。

- 相同 Stage 中的 Jobs 會並行執行
- 任一 Job 失敗，那麼 Stage 失敗，Pipeline 失敗
- 相同 Stage 中的 Jobs 都執行成功時，該 Stage 成功

### Runner

Runner 負責運行 job。需要先架好 Runner，並在 Gitlab 上登記，當 pipeline 運行時，Gitlab 會指派 jobs 給可使用的 runner。

#### 分為三種類型

- [Shared runners](https://docs.gitlab.com/ee/ci/runners/runners_scope.html#shared-runners) are available to all groups and projects in a GitLab instance.
- [Group runners](https://docs.gitlab.com/ee/ci/runners/runners_scope.html#group-runners) are available to all projects and subgroups in a group. It process jobs by using FIFO.
- [Specific runners](https://docs.gitlab.com/ee/ci/runners/runners_scope.html#specific-runners) are associated with specific projects. Typically, specific runners are used for one project at a time. It process jobs by using FIFO.

## Reference

- https://www.bmc.com/blogs/continuous-delivery-continuous-deployment-continuous-integration-whats-difference/
- https://blog.tienyulin.com/ci-cd-concept/
- http://myblog-maurice.blogspot.com/2021/01/cicd.html
- https://docs.gitlab.com/ee/ci/runners/runners_scope.html
