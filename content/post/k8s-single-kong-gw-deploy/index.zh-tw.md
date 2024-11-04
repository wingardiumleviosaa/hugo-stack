---
title: '[kong] Single Gateway Deployment on Kubernetes'
tags:
  - Kong
  - Kubernetes
categories:
  - Kubernetes
date: 2024-03-16 16:44:00
slug: single-kong-gw-deploy-on-kubernetes
---

### TL; DR
Kong 有不同部署架構，本文僅示範單一的 kong gateway 節點在 Kubernetes 上的安裝方式。透過官方的 helm chart 部署，包含 kong admin、 kong proxy 以及原生的 Kong Admin UI - Kong Manager。

<!--more-->

![](https://imgur.com/8eEb5j0.png)

### 安裝步驟

1. 建立 namesapce

```
kubectl create ns kong
```

2. 建立 postgresql database

```yaml
# postgre-values.yaml
image:
  tag: 10.23.0 # 11 版後 kong 不相容
global:
  storageClass: "rook-ceph-block"
  postgresql:
    auth:
      postgresPassword: "kong"
      username: "kong"
      password: "kong"
      database: "kong"

```

```
helm install kong-pg -f postgre-values.yaml bitnami/postgresql -n kong
```

3. 建立 kong (含 kong gateway & kong manager)

主要修改 kong-values.yaml 的項目如下：

- 設定外部 PostgreSQL Database  
- 關閉 Ingress Controller，如果要建立 ingress 資源，統一使用 Ingress Nginx Controller  
- 關閉所有開啟的服務的 https port，統一使用 http，並透過 Ingress 加上原先已存在的 Cert-Manager Certificate 暴露服務  

```yaml
# kong-values.yaml
env:
  database: "postgres"
  pg_host: "kong-pg-postgresql.kong.svc.cluster.local"
  pg_port: 5432
  pg_user: kong
  pg_password: kong
  pg_database: kong

# 中間略

admin:
  enabled: true
  type: ClusterIP
  loadBalancerClass:
  annotations: {}
  labels: {}
  http:
    enabled: true
    servicePort: 8001
    containerPort: 8001
    parameters: []
  tls:
    enabled: false
    servicePort: 8444
    containerPort: 8444
    parameters:
    - http2
    client:
      caBundle: ""
      secretName: ""
  ingress:
    enabled: true
    ingressClassName: nginx
    # TLS secret name.
    tls: domain-cert-sdsp-stg.com-prod
    # Ingress hostname
    hostname: kong-admin.sdsp-stg.com
    annotations: {}
    # Ingress path.
    path: /
    pathType: ImplementationSpecific

# 中間略 

proxy:
  enabled: true
  type: ClusterIP
  loadBalancerClass:
  nameOverride: ""
  annotations: {}
  labels:
    enable-metrics: "true"
  http:
    enabled: true
    servicePort: 80
    containerPort: 8000
    parameters: []
  tls:
    enabled: false
    servicePort: 443
    containerPort: 8443
    parameters:
    - http2
    appProtocol: ""
  stream: []
  ingress:
    enabled: true
    ingressClassName: nginx
    annotations: {}
    labels: {}
    hostname: kong.sdsp-stg.com
    path: /
    pathType: ImplementationSpecific
    hosts: []
    # TLS secret(s)
    tls: domain-cert-sdsp-stg.com-prod

# 中間略 

ingressController:
  enabled: false

# 中間略 

manager:
  enabled: true
  type: ClusterIP
  loadBalancerClass:
  annotations: {}
  labels: {}
  http:
    enabled: true
    servicePort: 8002
    containerPort: 8002
    parameters: []
  tls:
    enabled: false
    servicePort: 8445
    containerPort: 8445
    parameters:
    - http2
  ingress:
    enabled: true
    ingressClassName: nginx
    tls: domain-cert-sdsp-stg.com-prod
    hostname: kong-manager.sdsp-stg.com
    annotations: {}
    path: /
    pathType: ImplementationSpecific


```

```
helm repo add kong https://charts.konghq.com
helm install kong-dev -f kong-values.yaml kong/kong -n kong
```