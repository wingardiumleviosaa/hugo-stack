---
title: '使用 istio operator 安裝 Istio v1.11'
categories: ["Kubernetes"]
tags: ["Kubernetes", "Istio"]
date: 2021-10-26 20:27:00
slug: kubernets-install-istio
---
## 下載 Istio
<!--more-->
### 下載資源
用自動化工具下載並提取最新版本（Linux 或 macOS）：
```
$ curl -L https://istio.io/downloadIstio | sh -
```
或是用指定參數下載指定的、不同處理器體系的版本。例如，下載 x86_64 架構的、1.6.8 版本的 Istio ，運行：
 ```
$ curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.6.8 TARGET_ARCH=x86_64 sh -
```
### 進入 Istio 包目錄
```
$ cd istio-1.11.4
```
安裝目錄包含： 
  - `samples/` 目錄下的示例應用程序
  - `bin/` 目錄下的 `[istioctl](https://www.bookstack.cn/read/istio-1.11-zh/af90c7c768b11bf8.md)` 客户端二進制文件

### 設定 istioctl
將 `istioctl` 客户端加入執行路径（Linux or macOS）:
```
$ export PATH=$PWD/bin:$PATH
```
    

## 部署 Istio Operator

```bash
$ istioctl operator init
```

此命令運行 Operator 在 istio-operator 命名空間中創建以下資源：

- Operator 自定義資源定義（CRD）
- Operator 控制器的 deployment 對象
- 一個用來訪問 Operator 指標的服務
- Istio Operator 運行必須的 RBAC 規則

查看創建的資源
```
kubectl get all -n istio-operator
```

## 安裝 Istio

可以依據 profile 安裝指定的 istio 套件

![](https://imgur.com/0Nzav1i.png)

```yaml
$ kubectl create ns istio-system
$ kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istiocontrolplane
spec:
  profile: demo
EOF
```
各種 profile 的 yaml 檔放置在 `./manifests/profiles` 下，可依照需求改內容參數。並使用 `$ kubectl apply -f xxx.yaml` 部屬。

## 安裝 Addons Component
不同於以往 1.6 以前的版本的 istio yaml 檔可直接指定 addonComponents，在 1.11 版如果要安裝 Kiali、Jaeger 等 addon component，則需要另外部屬。
![](https://imgur.com/ys9rKJn.png)
```
# 一次部屬所有 addons
$ kubectl apply -f samples/addons

# 單獨指定套件部屬
$ kubectl apply -f samples/addons/kiali.yaml
```

## Resource
- https://preliminary.istio.io/latest/zh/docs/setup/install/operator/
- https://istio.io/latest/docs/setup/getting-started/