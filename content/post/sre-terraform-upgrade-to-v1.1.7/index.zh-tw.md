---
title: Terraform 從 0.14 升級到 1.1.7 問題排查
tags:
  - Terraform
categories:
  - SRE
date: 2022-03-22 17:13:00
slug: terraform-upgrade-to-1-1-7
---

手上有 Terraform 0.14 版跑的腳本，最近發現 Terraform 已經升級到 1.1.7 了，便打算在升級的環境下，跑 0.14 版跑成功的腳本，看看是否有誤，紀錄一下遇到的問題以及解法。

<!--more-->

## Terraform 安裝

```bash
yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y install terraform
terraform version
```

![](https://imgur.com/c2TMsrQ.png)

## 問題

### Error: Kubernetes cluster unreachable: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable

![](https://imgur.com/WdJIg75.png)

加上 `KUBE_CONFIG_PATH` 變數解決。

```
export KUBE_CONFIG_PATH=$HOME/.kube/config
```

### Error: failed to download "xxxx"

![](https://imgur.com/NC8nl7w.png)

安裝 helm 並加入相對應的 repo

```bash=
[ -f /usr/local/bin/helm ] || (cd /tmp && curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod 755 get_helm.sh && /tmp/get_helm.sh)
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
```

### Error: Failed to create Ingress 'xxxingress' because: the server could not find the requested resource (post ingresses.extensions)

![](https://imgur.com/slalKPW.png)

將原先的 `kubernetes_ingress` provider 改成 `kubernetes_ingress_v1`，另外 service name & port 的參數格式有變，請參考[官網](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/ingress_v1)。

### Warning: Helm release "xxx" was created but has a failed status. Use the `helm` command to investigate the error, correct it, then run Terraform again.

基本上 terraform 安裝沒什麼問題，直接下 kubectl get pod 在相對應的 namespace 查看出問題的 pod 是哪些，並進一步 debug。
