---
title: '已登入 harbor 但 kubelet 仍會 ImagePullBackOff'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-04-14 19:33:00
slug: image-pull-backoff-after-harbor-login
---
在 kubernetes 環境上拉取私有鏡像倉庫 harbor 的 image 時，一直卡在 ImagePullBackOff 的狀態，decribe pod 發現是權限問題導致拉取失敗。 

<!--more-->

## 狀況說明
錯誤訊息如下

![](https://imgur.com/2QcmPnq.png)

```
Failed to pull image "10.1.5.142:4433/test/findkpsn:9d2e44d2": rpc error: code = Unknown desc = Error response from daemon: unauthorized: unauthorized to access repository: test/findkpsn, action: pull: unauthorized to access repository: test/findkpsn, action: pull
```
**然而實際上在本機上已經 docker login 成功過了，也可以直接使用 docker pull 拉取，但透過 k8s 拉取仍會失敗。**

![](https://imgur.com/n9aBDLP.png)

## debug 思路
1. 確認在同樣 repo 的 project 下的其他 image 是否也發生同樣的情況
		是，同樣 repo 的 project 的其他 image 也相同。
2. 確認在不同的 repo 是否也發生同樣的情況
		否，其他 repo 能正常夠過 kubectl 拉取，應能推斷部屬環境上沒問題。

## 原因
結果是因為沒有把 project 公開 =__=

![](https://imgur.com/WTWv7l6.png)

## 意外發現
仍然還是可以讓 project 維持在私有的狀況下，透過 kubectl 拉取。只要在定義資源時，加上 `imagePullSecrets` 的屬性，值指定為欲創建資源的 namespace 下的 kubernetes.io/dockerconfigjson 的 secret，即可拉取成功。

### 創建 docker-registry secret
```
kubectl create secret docker-registry <secretName> \
--docker-server=DOCKER_REGISTRY_SERVER \
--docker-username=DOCKER_USER \
--docker-password=DOCKER_PASSWORD -n <NAMESPACE>
```

### deployment 部屬檔
在 spec.template.spec 下新增 imagePullSecrets
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
  namespace: test
  labels:
    app: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: test
        image: 10.1.5.142:4433/test/test/findkpsn:9d2e44d2
      imagePullSecrets:
      - name: harbor
```

### 重新佈署

![](https://imgur.com/STQZ44K.png)

加上 imagePullSecrets 後，就可以成功拉取私有專案的鏡像了!

## Reference
- https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod