---
title: Terraform Connection to Kuberentes Refused Error
tags:
  - Kubernetes
  - Terraform
categories:
  - SRE
date: 2021-08-29 21:40:00
slug: terraform-connection-refused-error
---

## 問題

在 terrform apply 的時候一直卡在 `Error: Post "http://localhost/api/v1/namespaces": dial tcp [::1]:80: connect: connection refused` 的錯誤

<!--more-->

![](https://imgur.com/vQer935.png)

## 原因

看起來是沒有正確的抓到 Provider 裡面設的 kube config file

原本的腳本如下

```terraform
// main.tf
provider "helm" {
  kubernetes {
    config_path = "/root/.kube/config"
  }
}
```

</br>

```terraform
// kubernetes.tf
provider "kubernetes" {}
```

## 解法

修改腳本如下

```terraform
// main.tf
provider "kubernetes" {
    host = "https://10.1.5.140:8443"

    client_certificate     = file("~/.kube/client.pem")
    client_key             = file("~/.kube/client-key.pem")
    cluster_ca_certificate = file("~/.kube/ca.pem")
}
```

</br>

```terraform
// kubernetes.tf
// 只能有一個 provider，否則會有 Error: Duplicate provider configuration 的錯誤
```

重新佈署後錯誤碼就莫名其妙變了:

```bash
[root@master1 terraform]# terraform apply
helm_release.metrics-server[0]: Refreshing state... [id=metrics-server]

Error: Kubernetes cluster unreachable: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
```

![](https://imgur.com/gNmDHIo.png)

照指示加上 環境變數

```bash
export KUBE_CONFIG_PATH=/root/.kube/config
```

就莫名其妙成功了 = =

## 補充:

- k8s provider 說明
  https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs#credentials-config
- k8s cluster ca
  https://blog.csdn.net/ll837448792/article/details/103658502

### 配置 TLS 連接

1. 查看 kubectl 配置文件，裡面記錄了三個證書和 API server 的地址：

```yaml
[root@testm ~]# cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CR......
    server: https://10.1.5.145:8443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CR......
    client-key-data: LS0tLS1CRUdJTiBSU0......
```

2. 匯出金鑰及證書

```bash
[root@testm .kube]# export clientcert=$(grep client-cert ~/.kube/config | cut -d" " -f 6)
[root@testm .kube]# echo $clientcert
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0=......
[root@testm .kube]# export clientkey=$(grep client-key-data ~/.kube/config | cut -d" " -f 6)
[root@testm .kube]# echo $clientkey
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSB......
[root@testm .kube]# export ca=$(grep certificate-authority-data ~/.kube/config | cut -d" " -f 6)
[root@testm .kube]# echo $ca
LS0tLS1CRUdJTiBDRVJUSUZJQ0FU......
[root@testm .kube]# echo $clientcert | base64 -d > ./client.pem
[root@testm .kube]# echo $clientkey | base64 -d > ./client-key.pem
[root@testm .kube]# echo $ca | base64 -d > ./ca.pem
```

3. 從配置文件中讀取 server 地址：

```bash
[root@testm ~]# kubectl config view | grep server
    server: https://10.1.5.145:8443
```

4. 使用 curl 和剛剛加密的密鑰文件來訪問 API server：

```bash
curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem https://10.1.5.145:8443/api/v1/pods

{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "387199"
  },
  "items": [
    {
      "metadata": {
        "name": "ingress-nginx-controller-5c5bf8c854-7pcf7",
        "generateName": "ingress-nginx-controller-5c5bf8c854-",
        "namespace": "ingress-nginx",
        "uid": "f84b09e4-7d7d-40bf-ae66-f2eb72ab7a59",
        "resourceVersion": "104900",
        "creationTimestamp": "2022-02-09T05:42:28Z",
.
.
.
```
