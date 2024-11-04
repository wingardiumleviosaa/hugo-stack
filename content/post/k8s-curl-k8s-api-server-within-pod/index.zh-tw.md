---
title: '在容器裡 curl Kubernetes API server'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-03-31 21:57:12
slug: curl-kubernetes-api-server-within-pod
---
連接 k8s 的 api-server 有三種方式：
1. Kubernetes Node 通過 kubectl proxy 中轉連接
2. 通過授權驗證直接連接，例如 kubectl 和各種 client 就是這種情況
3. 容器內部通過 ServiceAccount 連接

本文以第三種情況作範例。

<!--more-->

## Kubernetes API Server
在 Kubernetes 集群被創建時，預設會在 default namespace 中創建 kubernetes 的服務，用於訪問 Kubernetes apiserver。因此，Pod 之間可以直接使用 kubernetes.default.svc 主機名來查詢 API server。

![](https://imgur.com/DNlD28U.png)

## Service Account
ServiceAccount 是給執行在 Pod 的程式使用的身份認證，給 Pod 容器的程式訪問 API Server 時使用；ServiceAccount 僅侷限它所在的 namespace，每個 namespace 建立時都會自動建立一個 default service account；建立 Pod 時，如果沒有指定 Service Account，Pod 則會使用 default Service Account。

![](https://imgur.com/YPM8C0A.png)

## Service Account Secret
SA 對應的 Secret 會自動掛載到 Pod 的 /var/run/secrets/kubernetes.io/serviceaccount/ 目錄中(包含 token、ca.crt、namespace)。

![](https://imgur.com/cDSpKGI.png)

## 創建 Role & Role Binding
如果直接使用預設的 sa 訪問 api server 會遇到權限不足的問題

![](https://imgur.com/vyPzlfl.png)

此時需要建立角色開放存取 api 指定路徑的權限並綁定角色到 SA 上
```yaml
# role&binding.yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: default-role
  namespace: converg-it
rules:
  - apiGroups: [""]
    resources:
      - pods
      - pods/log
    verbs:
      - get
      - list
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default-roldbinding
  namespace: converg-it
subjects:
  - kind: ServiceAccount
    name: default
roleRef:
  kind: Role
  name: default-role
  apiGroup: rbac.authorization.k8s.io
```
```
kubectl -n converg-it apply -f role&binding.yaml
role.rbac.authorization.k8s.io/default-role created
rolebinding.rbac.authorization.k8s.io/default-roldbinding created
```

## curl API
進入容器環境
```
kubectl exec -it -n converg-it converg-it-adapter-oracle-8575f54dc6-7lnv6 -- bash
```
get target API
```
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" -X GET https://kubernetes.default.svc/api/v1/namespaces/converg-it/pods/converg-it-adapter-oracle-8575f54dc6-7lnv6/log?sinceSeconds=300
```

![](https://imgur.com/1aEtzgL.png)

## Reference
- https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/