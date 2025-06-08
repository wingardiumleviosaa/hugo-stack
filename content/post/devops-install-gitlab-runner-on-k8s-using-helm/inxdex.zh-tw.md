---
title: 在 Kubernetes 上使用 helm 建立 GitLab Runner
tags:
  - GitLab
  - CI/CD
categories:
  - DevOps
date: 2022-04-10 16:36:00
slug: install-gitlab-runner-on-k8s-by-helm
---

GitLab Runner 是一個獨立的程序，可以用以下三種方式安裝，請參考[官網教學](https://docs.gitlab.com/runner/install/)。

- GitLab Official Repositories RPM/deb packages
- Using Binaries
- Using Containers

<!--more-->

承前一篇介紹，Runner 有 3 種類型，所有項目共享的 Shared Runner、指定 Group 共享的 Group Runner 和單個項目獨占的 Specific Runner，各有不同的 registration token。

- Shared Runner：在 Admin Area > Runners 中註冊
- Group Runner：在指定的 group 的 Settings > CI/CD > Runners 中註冊
- Specific Runner：在各自的 project 下的 Settings > CI/CD > Runners 中註冊

本篇記錄使用 helm chart 的方式安裝 Runner 於 Kubernetes 上的過程。

## Prerequisite

- GitLab Server (目前使用 14.9.2 版，用官方提供的 docker-compose 腳本安裝，可參考之前的[文章記錄](https://ulahsieh.netlify.app/p/docker-compose-gitlab-https/))
- Kubernetes 1.4+ (示範環境為 1.20.10)
- Helm (2 或 3 皆可，本文使用 helm3)
- GitLab server 與 Kubernetes 集群能互通

## 準備 helm chart

```
helm repo add gitlab https://charts.gitlab.io
  helm repo list
  NAME  	URL
  gitlab	https://charts.gitlab.io/
```

### 建立自簽憑證的 Kubernetes Secret

將建立 gitlab 時創建的自簽憑證創建為 secret 資源，該憑證的檔名應採用 `<gitlab.hostname>.crt` 格式。

```
kubectl create secret generic <SECRET_NAME> \
  --namespace <NAMESPACE> \
  --from-file=<CERTIFICATE_FILENAME>
```

- <NAMESPACE> 為安裝 GitLab Runner 的 Kubernetes 命名空間。
- <SECRET_NAME> 是 Kubernetes Secret 資源名稱。(例如：gitlabcert。)
- <CERTIFICATE_FILENAME> 是當前目錄中將被導入密鑰的證書的文件名。(例如 `10.1.5.142.crt`。)

### 建立存取私有倉庫的 Secret

```
kubectl create secret docker-registry <SECRET_NAME> \
  --namespace <NAMESPACE> \
  --docker-server="https://10.1.5.142:4433/" \
  --docker-username="admin" \
  --docker-password="Harbor12345"
```

### 準備自訂義配置文件

`values.yaml` 的詳細配置可以在 [GitLab Runner Helm Chart](https://gitlab.com/gitlab-org/charts/gitlab-runner/-/blob/main/values.yaml) 查看。
以下是此次演示所用的 `values.yaml` 內容：

```yaml
imagePullPolicy: IfNotPresent
gitlabUrl: "https://10.1.5.142"
runnerRegistrationToken: "xWKuLU_jk_PDvs2k9auF"
certsSecretName: "gitlabcert"
concurrent: 10
checkInterval: 30
logLevel: "debug"

rbac:
  create: false

metrics:
  enabled: false

runners:
  name: "kubernetes-runner"
  tags: "kubernetes,runner"
  executor: "kubernetes"
  config: |
    [[runners]] 
      [runners.kubernetes]
        namespace = "{{.Release.Namespace}}"
        image = "alpine:latest"
        pull_policy = "if-not-present"
        image_pull_secrets = ["harbor"]
        privileged = true
        [[runners.kubernetes.volumes.host_path]]
            name = "docker"
            mount_path = "/var/run/docker.sock"
            host_path = "/var/run/docker.sock"
```

其中：

- certsSecretName：指定 GitLab 的自簽憑證，以 kubernetes secret 的方式建立
- concurrent：並行運行 Job 的最大 pod 數，部署在 Kubernetes 上的單個 GitLab Runner 能夠通過自動啟動額外的 Runner pod 來並行執行多個作業
- checkInterval：Gitlab 檢查新構建的時間間隔，以秒為單位，預設為 3。
- runner：runner 的配置内容，包括 name，tag 等等，這些内容建立後會以 config.toml 文件的形式包在 kubernetes configmap 之中；
- [runners.kubernetes] 表示 Kubernetes executor，會將 job 跑在 kubernetes pod 中。
- image：指定預設的 docker image 給 job pod，當 pipeline 沒有指定時便會使用該預設。
- 因為之後會需要使用 docker-in-docker 方式 (在 pipeline 中構建 docker 鏡像)，所以需要掛載 /var/run/docker.sock 到 gitlab runner pod 中，另外還要設置 privileged 為 true，否則會報無法連接 docker daemon 的錯誤。

{{< notice warning >}}
在生產環境中，使用特權模式運行 docker-in-docker 有一些注意事項，可以參考這篇 <a href="https://docs.gitlab.com/runner/executors/kubernetes.html#using-docker-in-your-builds">文檔</a>
{{< /notice >}}

### 開始建立

建立 namespace

```
kubectl create ns gitlab-runner
```

安裝

```
helm install --namespace gitlab-runner gitlab-runner -f values.yaml gitlab/gitlab-runner
  NAME: gitlab-runner
  LAST DEPLOYED: Mon Apr 11 14:27:15 2022
  NAMESPACE: gitlab-runner
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  Your GitLab Runner should now be registered against the GitLab instance reachable at: "https://10.1.5.142"

  Runner namespace "gitlab-runner" was found in runners.config template.
```

查看資源

```
kubectl get all -n gitlab-runner
  NAME                                              READY   STATUS    RESTARTS   AGE
  pod/gitlab-runner-gitlab-runner-899597bcb-nzjwc   1/1     Running   0          61m

  NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/gitlab-runner-gitlab-runner   1/1     1            1           61m

  NAME                                                    DESIRED   CURRENT   READY   AGE
  replicaset.apps/gitlab-runner-gitlab-runner-899597bcb   1         1         1       61m
```

查看 pod log，可以看到註冊成功的訊息。

```bash
 kubectl logs -f -n gitlab-runner gitlab-runner-gitlab-runner-899597bcb-nzjwc
  Registration attempt 1 of 30
  Runtime platform arch=amd64 os=linux pid=14 revision=d1f69508 version=14.9.0
  WARNING: Running in user-mode.
  WARNING: The user-mode requires you to manually start builds processing:
  WARNING: $ gitlab-runner run
  WARNING: Use sudo for system-mode:
  WARNING: $ sudo gitlab-runner...

  Merging configuration from template file "/configmaps/config.template.toml"
  Registering runner... succeeded                     runner=xWKuLU_j
  Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
  ......
```

重整 GitLab Runner 頁面，可以看到註冊成功的 runner 列表。

![](https://imgur.com/W9Spfwp.png)

## Reference

- https://docs.gitlab.com/runner/install/kubernetes.html
- https://docs.gitlab.com/runner/configuration/advanced-configuration.html
- https://docs.gitlab.com/runner/executors/kubernetes.html
