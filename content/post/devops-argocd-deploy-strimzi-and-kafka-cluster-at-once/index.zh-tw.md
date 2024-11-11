---
title: "使用 ArgoCD helm chart 自動化部署與管理 Strimzi Deployment (包含 nodepool 及 cluster)"
tags:
  - ArgoCD
  - Kafka
categories:
  - DevOps
  - ArgoCD
  - ApacheKafka
  - Kubernetes
date: 2024-10-18T14:35:17+08:00
slug: devops-argocd-deploy-strimzi-and-kafka-cluster-at-once
---

## TL; DR

紀錄如何使用 argocd 管理 srimzi 的部屬

## Prerequisite

- Kubernetes
- ArgoCD
- GitLab
- 建立 ArgoCD application 並已於 GitLab 設置 ArgoCD 的 Repository

## 放置部署檔案

目錄結構

```
kafka/
├── Chart.yaml
├── values.yaml
└── templates/
    └── kafka-nodepool.yaml
```

- Chart.yaml

```yaml
apiVersion: v2
appVersion: "3.8"
description: "Strimzi: Apache Kafka running on Kubernetes"
name: strimzi-kafka-operator
version: 0.43.0
icon: https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/main/documentation/logo/strimzi_logo.png
home: https://strimzi.io/
sources:
  - https://github.com/strimzi/strimzi-kafka-operator
dependencies:
  - name: strimzi-kafka-operator
    version: 0.43.0
    repository: oci://quay.io/strimzi-helm
  - name: kafka-ui
    version: 0.7.6
    repository: https://provectus.github.io/kafka-ui-charts
```

- values.yaml

```yaml
strimzi-kafka-operator:
  extraEnvs:
    - name: STRIMZI_LABELS_EXCLUSION_PATTERN
      value: "argocd.argoproj.io/instance"

clusterName: my-cluster

kafka:
  version: "3.8.0"
  metadataVersion: "3.8"
  replication:
    offsetsTopicFactor: 1
    transactionStateLogFactor: 3
    transactionStateLogMinIsr: 2
    defaultReplicationFactor: 3
    minInsyncReplicas: 2
  logRetention:
    hours: 2160
    bytes: -1
  listeners:
    - name: plain
      port: 9092
      type: internal
      tls: false
    - name: external
      port: 9094
      type: nodeport
      tls: false
      configuration:
        bootstrap:
          nodePort: 32100
        brokers:
          - broker: 0
            advertisedHost: kafka.sdsp-stg.com
            advertisedPort: 8091
          - broker: 1
            advertisedHost: kafka.sdsp-stg.com
            advertisedPort: 8092
          - broker: 2
            advertisedHost: kafka.sdsp-stg.com
            advertisedPort: 8093

controllerNodePool:
  replicas: 3
  storage:
    size: "100Gi"
    deleteClaim: true
    class: ceph-block

brokerNodePool:
  replicas: 3
  storage:
    size: "200Gi"
    deleteClaim: true
    class: ceph-block

kafka-ui:
  yamlApplicationConfig:
    kafka:
      clusters:
        - name: stg
          bootstrapServers: my-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092
    auth:
      type: LOGIN_FORM
    spring:
      security:
        user:
          name: admin
          password: "!QAZxsw2"
    management:
      health:
        ldap:
          enabled: false

  ingress:
    enabled: true
    ingressClassName: nginx
    path: "/"
    pathType: Prefix
    host: kafka-ui.sdsp-stg.com
    tls:
      enabled: true
```

- templates/kafka-nodepool.yaml  
  透過自定義 helm template 的方式部署原本要手動透過 kubeapply YAML manifest 建立的 nodepool 以及 kafka cluster 資源。

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaNodePool
metadata:
  name: controller
  labels:
    strimzi.io/cluster: {{ .Values.clusterName }}
spec:
  replicas: {{ .Values.controllerNodePool.replicas }}
  roles:
    - controller
  storage:
    type: jbod
    volumes:
      - id: 0
        type: persistent-claim
        size: {{ .Values.controllerNodePool.storage.size }}
        kraftMetadata: shared
        deleteClaim: {{ .Values.controllerNodePool.storage.deleteClaim }}
        class: {{ .Values.controllerNodePool.storage.class }}

---

apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaNodePool
metadata:
  name: broker
  labels:
    strimzi.io/cluster: {{ .Values.clusterName }}
spec:
  replicas: {{ .Values.brokerNodePool.replicas }}
  roles:
    - broker
  storage:
    type: jbod
    volumes:
      - id: 0
        type: persistent-claim
        size: {{ .Values.brokerNodePool.storage.size }}
        kraftMetadata: shared
        deleteClaim: {{ .Values.brokerNodePool.storage.deleteClaim }}
        class: {{ .Values.brokerNodePool.storage.class }}

---

apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: {{ .Values.clusterName }}
  annotations:
    strimzi.io/node-pools: enabled
    strimzi.io/kraft: enabled
spec:
  kafka:
    version: {{ .Values.kafka.version }}
    metadataVersion: {{ .Values.kafka.metadataVersion | quote }}
    listeners:
      {{- range .Values.kafka.listeners }}
      - name: {{ .name }}
        port: {{ .port }}
        type: {{ .type }}
        tls: {{ .tls }}
        {{- if .configuration }}
        configuration:
          bootstrap:
            nodePort: {{ .configuration.bootstrap.nodePort }}
          brokers:
          {{- range .configuration.brokers }}
            - broker: {{ .broker }}
              advertisedHost: {{ .advertisedHost }}
              advertisedPort: {{ .advertisedPort }}
          {{- end }}
        {{- end }}
      {{- end }}
    config:
      offsets.topic.replication.factor: {{ .Values.kafka.replication.offsetsTopicFactor }}
      transaction.state.log.replication.factor: {{ .Values.kafka.replication.transactionStateLogFactor }}
      transaction.state.log.min.isr: {{ .Values.kafka.replication.transactionStateLogMinIsr }}
      default.replication.factor: {{ .Values.kafka.replication.defaultReplicationFactor }}
      min.insync.replicas: {{ .Values.kafka.replication.minInsyncReplicas }}
      log.retention.hours: {{ .Values.kafka.logRetention.hours }}
      log.retention.bytes: {{ .Values.kafka.logRetention.bytes }}
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

## 部署結果

## 問題紀錄

### PVC 無法自動創建

**錯誤訊息：**

由於 PVC 不存在，所以 Kafka broker 跟 controller 的 pod 狀態一直卡在 Pending。

**解決方式：**

將 NodePool 的 `deleteClaim` 設定為 `true` 。`deleteClaim` 設置決定了當 Kafka 叢集卸載時，是否要刪除 PVC。Strimzi 為了避免 PVC 被無意間刪除，因此沒有將 PVC 設為 Kafka Custom Resource 的擁有資源。然而，這導致 ArgoCD 認為 PVC 是應由應用創建的，但卻沒有擁有者引用，而將其刪除。所以設為 `true` 後，Strimzi 會自動更新 PVC 的擁有者引用，這樣 ArgoCD 就不會再嘗試刪除。

### KafkaNodePool 創建的 Service Account 沒有權限讀取 Kubernetes Node API

**錯誤訊息：**

```yaml
Message: nodes "bpzaw01k8s" is forbidden: User "system:serviceaccount:kafka:my-cluster-kafka" cannot get resource "nodes" in API group "" at the cluster scope. Received status: Status(apiVersion=v1, code=403, details=StatusDetails(causes=[], group=null, kind=nodes, name=bpzaw01k8s, retryAfterSeconds=null, uid=null, additionalProperties={}), kind=Status
```

**解決方式：**

```yaml
strimzi-kafka-operator:
  extraEnvs:
    - name: STRIMZI_LABELS_EXCLUSION_PATTERN
      value: "argocd.argoproj.io/instance"
```

ArgoCD 在管理資源時會注入標籤，例如 `argocd.argoproj.io/instance`，以便追踪資源的狀態。這些標籤可能導致 `Strimzi Operator` 誤判哪些資源需要管理或更新。所以加上這個設定讓 `Strimzi Operator` 正確地忽略了 `ArgoCD` 注入的標籤，並避免在角色和權限方面產生衝突。

## Reference

- https://github.com/strimzi/strimzi-kafka-operator/issues/7093
