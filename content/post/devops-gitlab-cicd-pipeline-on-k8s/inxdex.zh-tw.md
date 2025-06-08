---
title: 創建 GitLab CICD pipeline 完成自動部屬到 Kubernetes
author: Ula
tags:
  - GitLab
  - CI/CD
categories:
  - DevOps
date: 2022-04-15 18:18:00
slug: build-cicd-pipeline-and-cd-to-k8s
---

## 前言

在還沒接觸 CI/CD 時，一直有『這東西一定很難』的預設立場，直到開始撰寫第一個 .gitlab-ci.yml 時，心理認知的困難度仍沒消失。不過慶幸的是，網路上的教學真的很多，GitLab 社群也超給力的有著豐富的文檔跟範例。GitLab CI/CD 整體架構 (gitlab server、runner、excutor、pipeline、stage、job) 其實很單純，所以在實做時，可以把自動化需求切分，先從第一段 build 開始做，做成功後，再進階到 test 及 deploy。把困難的任務分段做，感覺就不那麼難以親近了！

<!--more-->

## 確認 CI/CD 需求

[前面文章](https://ulahsieh.netlify.app/p/install-gitlab-runner-on-k8s-by-helm/)已經建好了一個 runner，現在要實做 CI/CD pipeline。在開始之前要先確定開發者想透過 CICD 完成哪部分的自動化？以本次實做為例，我想要達成：

1. code 推上 GitLab 後，不須測試，因為在推上 repo 前，皆已在本地端測試完畢。
2. 自動依據專案中的 Dockerfile 檔案 build 成 image，並上傳到 registry，如 DockerHub 或私有倉庫
3. 自動依據專案中的 kubernetes.yaml 檔部屬到 kubernetes cluster 中

## .gitlab-ci.yml

GitLab CI/CD 透過放置在專案根目錄底下的 `.gitlab-ci.yml` 檔案來驅動，官網提供了許多 [Template](https://docs.gitlab.com/ee/ci/examples/#cicd-templates) 給初學者參考。
定義 pipeline 中會有哪些 stages (預計要有的構建階段)，依序設置。

```yaml
stages:
  - build
  - deploy
```

接著在 Stage 中設置一個到多個 Job，來描述該階段所需完成的工作。

```yaml
<jobName>:
  stage: <stageName>
  only:
    - <branchName>
  script:
    - echo "My first job"
```

其中

- stage：指定該 Job 屬於哪一個 Stage
- only：指定在哪個 Branch 觸發時才會執行
- script：需要執行的指令

## 配置 .gitlab-ci.yml

確定需求後就可以開始寫 .gitlab-ci.yml 檔。

### 先實做 build 階段

```yaml
stages:
  - build

variables:
  IMAGE_NAME: findkpsn
  CI_IMAGE: $CI_REGISTRY/test/$IMAGE_NAME:${CI_COMMIT_SHORT_SHA}

build-image:
  stage: build
  tags:
    - kubernetes
  image: docker
  variables:
    DOCKER_DRIVER: overlay2
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_IMAGE .
    - docker push $CI_IMAGE
```

其中：

- `$CI_REGISTRY`、`$CI_REGISTRY_USER`、`$CI_REGISTRY_PASSWORD` 變數紀錄在 gitlab > Admin Area > CI/CD > Variables 中
  ![](https://imgur.com/gPoZtZm.png)
- `${CI_COMMIT_SHORT_SHA}` 為 GitLab 預設的環境變數，表示 Git Repo 目前所在的 Commit Hash Code 的前 8 字元。
- `DOCKER_DRIVER`：指定運行的 docker driver，使用 [overlayfs](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) 以增進 performance。但 docker 預設的 Storage Driver 就是 overlay2，所以這邊可以省略。

### 在跑的過程中遇到以下問題

**ERROR: Job failed (system failure): prepare environment: setting up credentials: secrets is forbidden: User "system:serviceaccount:gitlab-runner:default" cannot create resource "secrets" in API group "" in the namespace "gitlab-runner".**

![](https://imgur.com/DfE2rzH.png)

原因是因為使用 helm chart 建立 Runner 時，在指定的 values.yaml 設定檔中，應該把 rbac: create: 的值設為 true，在建立時便會自動幫該 namespace 設定對應 service account。

**ERROR: Job failed: command terminated with exit code 1 (Error response from daemon: Get "https://10.1.5.142:4433/v2/": x509: certificate signed by unknown authority)**

![](https://imgur.com/9zCxj1K.png)

原因是因為在 pod 所在的 worker node 的 docker 沒有設置對應的 registry 的證書資訊。在 worker node 中的 `/etc/docker/cert.d/` 中，建立針對該倉庫 URL 的資料夾 (如 10.1.5.142:4433)，並將倉庫的證書、金鑰、ca 證書放置其中。

![](https://imgur.com/rjxu0Fl.png)

**ERROR: Job failed: command terminated with exit code 1 (error obtaining VCS status: exec: "git": executable file not found in $PATH)**

![](https://imgur.com/lJzTfk9.png)

{{< notice info >}}
其實執行到這步驟，設定的 Git Lab CI 檔已經可以正常運作了！這個錯誤訊息已經是 docker build image 階段的錯誤了，意即 Dockerfile 沒寫好，不過順便還是在這篇文章記錄一下。
{{< /notice >}}

在 go build 的過程中如果有 import 從 git repo 上的第三方程式包的話，則需要在 go build 環境中有 `git`。只要在 Dockerfile 中加上 `RUN apk add --no-cache git` 即可解決。

### 後實做 deploy 階段

補齊 deploy stage

```yaml
stages:
  - build
  - deploy

variables:
  IMAGE_NAME: findkpsn
  CI_IMAGE: $CI_REGISTRY/test/$IMAGE_NAME:${CI_COMMIT_SHORT_SHA}

build-image:
  stage: build
  tags:
    - kubernetes
  image: docker
  variables:
    DOCKER_DRIVER: overlay2
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_IMAGE .
    - docker push $CI_IMAGE

deploy-to-k8s:
  stage: deploy
  tags:
    - kubernetes
  image: bitnami/kubectl:1.20.15
  script:
    - kubectl version
    - sed -i "s/<VERSION>/${CI_COMMIT_SHORT_SHA}/g" kubernetes.yaml
    - kubectl create ns findkpsn
    - kubectl apply -f kubernetes.yaml
```

### kubernetes.yaml 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  name: findkpsn
  namespace: findkpsn
  labels:
    app: findkpsn
spec:
  type: ClusterIP
  selector:
    app: findkpsn
  ports:
    - name: findkpsn
      port: 8080
      protocol: TCP
      targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: findkpsn
  namespace: findkpsn
  labels:
    app: findkpsn
spec:
  replicas: 1
  selector:
    matchLabels:
      app: findkpsn
  template:
    metadata:
      labels:
        app: findkpsn
    spec:
      containers:
        - name: findkpsn
          image: 10.1.5.142:4433/test/findkpsn:<VERSION>
          imagePullPolicy: Always
```

### 在跑的過程中遇到以下問題

**Error from server (Forbidden): error when retrieving current configuration of: ... services "xxx" is forbidden: User "system:serviceaccount:gitlab-runner:default" cannot get resource "services" in API group "" in the namespace "xxx"**  
遇到前面 helm 創建 gitlab runner 時的服務帳戶沒有創建 service 資源的功能。直接使用下面 command 賦予 cluster-admin 的權限給該角色。

```
kubectl create clusterrolebinding default --clusterrole=cluster-admin --group=system:serviceaccounts --namespace=gitlab-runner
	clusterrolebinding.rbac.authorization.k8s.io/default created
```

或是在創建 runner 時將 rule 設為全部，並開啟 `clusterWideAcess` 的權限

```yaml
rbac:
  create: true
  rules:
    - apiGroups: [""]
      resources: ["*"]
      verbs: ["*"]
  clusterWideAccess: true
```

## 成功

![](https://imgur.com/L4cd5Pu.png)

![](https://imgur.com/fTBS3Ju.png)

![](https://imgur.com/ItXsMzH.png)

順便觀察 Runner 在工作的過程會有很多 pod 被建立，隨著構建工作結束後 Runner 會自動刪除這些 pod。

![](https://imgur.com/wn5Vd3P.png)

## Reference

- https://docs.gitlab.com/runner/install/kubernetes.html
- 感謝 https://marsping.gitlab.io/GitLabCICD/section1.html 給的勇氣，研發就是不停踩坑再爬起來的過程 XDD
