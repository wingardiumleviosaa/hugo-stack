---
title: Install Pure Prometheus Stack by Kustomize on Kubernetes
tags:
  - Kubernetes
  - Prometheus
categories:
  - Kubernetes
date: 2024-11-27 18:45:00
slug: k8s-install-pure-prometheus-stack-by-kustomize
---

## TL;DR

使用 ArgoCD 透過 kustomize 部署 Prometheus、Push Gateway、Grafana 及 Alertmanager。

## Highlight

在部署時遇到幾個問題，紀錄相對應的解法並在後面提供部署的 YAML 檔。

1. Prometheus persistent 遇到權限錯誤

   錯誤訊息如下：

   ```yaml
   level=error component=activeQueryTracker msg="Error opening query log file" file=/prometheus/queries.active err="open /prometheus/queries.active: permission denied"
   ```

   需要加上 `securityContext`設定項，使用 root 的身分執行。

   ```yaml
   securityContext:
     runAsUser: 0
   ```

2. Grafana persistent 遇到權限錯誤

   錯誤訊息如下：

   ```yaml
   mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied
   ```

   需要加上 `securityContext`設定項，使用指定身分執行。

   ```yaml
   securityContext:
     fsGroup: 472
     supplementalGroups:
       - 0
   ```

3. configmap 更新後無法自動替換 pod

   原因是因為原本的 pvc 是設定 ReadWriteOnce，故需將 PVC 改為 ReadWriteMany 並使用 支援 RWX 的 storageclass (CephFS)。

   ```yaml
   spec:
     accessModes:
       - ReadWriteMany
     resources:
       requests:
         storage: 10Gi
     storageClassName: ceph-filesystem
   ```

   補充如果沒有設置對的 storageclass 的話，會報錯

   ```yaml
   Warning  ProvisioningFailed    11s (x9 over 2m19s)   rbd.csi.ceph.com_csi-rbdplugin-provisioner-86b6cb5559-cb6vh_dab2e893-3479-41d3-bd89-2efdacd50f2b  failed to provision volume with StorageClass "ceph-block": rpc error: code = InvalidArgument desc = multi node access modes are only supported on rbd `block` type volumes
   ```

   以下截自[官網](https://rook.io/docs/rook/v1.10/Troubleshooting/ceph-csi-common-issues/#block-rbd)說明為什麼 ReadWriteMany 需要使用 CephFS。

{{< quote source="Rook Official Documentation" url="https://rook.io/docs/rook/v1.10/Troubleshooting/ceph-csi-common-issues/#block-rbd">}}

**Block (RBD)**

If you are mounting block volumes (usually RWO), these are referred to as `RBD` volumes in Ceph. See the sections below for RBD if you are having block volume issues.

**Shared Filesystem (CephFS)**

If you are mounting shared filesystem volumes (usually RWX), these are referred to as `CephFS` volumes in Ceph. See the sections below for CephFS if you are having filesystem volume issues.

{{< /quote >}}

## 部署檔案

### kustomization.yaml

{{< detail-tag "CLICK ME" >}}

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

configMapGenerator:
  - files:
      - alertmanager.yml
    name: alertmanager-config
  - files:
      - prometheus.yml
    name: prometheus-config
  - files:
      - rules.yaml
    name: prometheus-rules

resources:
  - deployment.yaml
  - httproute.yaml
```

{{< /detail-tag >}}

### deployment.yaml

{{< detail-tag "CLICK ME" >}}

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: ceph-filesystem

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: ceph-block

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus:v2.53.0
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus"
          volumeMounts:
            - name: prometheus-config
              mountPath: /etc/prometheus
            - mountPath: "/prometheus"
              subPath: prometheus
              name: data
            - name: prometheus-rules
              mountPath: /etc/prometheus/rules
              readOnly: true
          ports:
            - containerPort: 9090
      securityContext:
        runAsUser: 0
      volumes:
        - name: prometheus-config
          configMap:
            name: prometheus-config
        - name: data
          persistentVolumeClaim:
            claimName: prometheus-pvc
        - name: prometheus-rules
          configMap:
            name: prometheus-rules

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pushgateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pushgateway
  template:
    metadata:
      labels:
        app: pushgateway
    spec:
      containers:
        - name: pushgateway
          image: prom/pushgateway:v1.10.0
          ports:
            - containerPort: 9091

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
          - 0
      containers:
        - name: grafana
          image: grafana/grafana:11.0.0
          env:
            - name: GF_SECURITY_ADMIN_USER
              value: "admin"
            - name: GF_SECURITY_ADMIN_PASSWORD
              value: "!QAZxsw2"
          volumeMounts:
            - name: grafana-data
              mountPath: /var/lib/grafana
          ports:
            - containerPort: 3000
      volumes:
        - name: grafana-data
          persistentVolumeClaim:
            claimName: grafana-pvc

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alertmanager
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alertmanager
  template:
    metadata:
      labels:
        app: alertmanager
    spec:
      containers:
        - name: alertmanager
          image: prom/alertmanager:v0.27.0
          args:
            - "--config.file=/etc/alertmanager/alertmanager.yml"
          ports:
            - containerPort: 9093
          volumeMounts:
            - name: alertmanager-config
              mountPath: /etc/alertmanager
      volumes:
        - name: alertmanager-config
          configMap:
            name: alertmanager-config

---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  ports:
    - port: 9090
      targetPort: 9090
  selector:
    app: prometheus

---
apiVersion: v1
kind: Service
metadata:
  name: pushgateway
spec:
  ports:
    - port: 9091
      targetPort: 9091
  selector:
    app: pushgateway

---
apiVersion: v1
kind: Service
metadata:
  name: grafana
spec:
  ports:
    - port: 3000
      targetPort: 3000
  selector:
    app: grafana

---
apiVersion: v1
kind: Service
metadata:
  name: alertmanager
spec:
  ports:
    - port: 9093
      targetPort: 9093
  selector:
    app: alertmanager
```

{{< /detail-tag >}}

### httproute.yaml

{{< detail-tag "CLICK ME" >}}

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: prometheus-route
spec:
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - "prom-flink.sdsp-dev.com"
  rules:
    - backendRefs:
        - group: ""
          kind: Service
          name: prometheus
          port: 9090
          weight: 1
      matches:
        - path:
            type: PathPrefix
            value: /

---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: pushgateway-route
spec:
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - "prom-push.sdsp-dev.com"
  rules:
    - backendRefs:
        - group: ""
          kind: Service
          name: pushgateway
          port: 9091
          weight: 1
      matches:
        - path:
            type: PathPrefix
            value: /

---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: grafana-route
spec:
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - "grafana-flink.sdsp-dev.com"
  rules:
    - backendRefs:
        - group: ""
          kind: Service
          name: grafana
          port: 3000
          weight: 1
      matches:
        - path:
            type: PathPrefix
            value: /

---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: alertmanager-route
spec:
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - "alert-flink.sdsp-dev.com"
  rules:
    - backendRefs:
        - group: ""
          kind: Service
          name: alertmanager
          port: 9093
          weight: 1
      matches:
        - path:
            type: PathPrefix
            value: /
```

{{< /detail-tag >}}

### prometheus.yml

{{< detail-tag "CLICK ME" >}}

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
rule_files:
  - "/etc/prometheus/rules/*.yaml"
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "pushgateway"
    honor_labels: true
    static_configs:
      - targets: [pushgateway:9091]
```

{{< /detail-tag >}}

### alertmanager.yml

{{< detail-tag "CLICK ME" >}}

```yaml
global:
  resolve_timeout: 5m
route:
  group_by: ["namespace"]
  group_wait: 3s
  group_interval: 5m
  repeat_interval: 10m
  receiver: "null"
  routes:
    - receiver: "null"
      matchers:
        - "alertname = Watchdog"
    - receiver: "null"
      matchers:
        - "alertname = InfoInhibitor"
    - receiver: "critical-warn-receiver"
      matchers:
        - "severity =~ critical|warning"
receivers:
  - name: "null"
  - name: "critical-warn-receiver"
    msteams_configs:
      - webhook_url: "https://prod2-08.southeastasia.logic.azure.com:443/workflows/8320d1a655114348ba880f0cc2fb5f16/triggers/manual/paths/invoke?api-version=2016-06-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=dBNbf2xwW28kg89uF8th54h4dZk_xwdnrIFzu9gWHaU"
    webhook_configs:
      - url: "https://alerta.sdsp-stg.com/api/webhooks/prometheus?api-key=OaksgBaMMIPDsp9XZEm6X-qel7ZwQAuxs2jYZQee"
        send_resolved: true
templates:
  - "/etc/alertmanager/config/*.tmpl"
```

{{< /detail-tag >}}

### rules.yml

{{< detail-tag "CLICK ME" >}}

```yaml
groups:
  - name: MySQLStatsAlert
    rules:
      - alert: MySQL is down
        expr: mysql_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} MySQL is down"
          description: "MySQL database is down. This requires immediate action!"
```

{{< /detail-tag >}}
