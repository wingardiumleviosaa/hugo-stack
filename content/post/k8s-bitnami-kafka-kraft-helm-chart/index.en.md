---
title: '[Kubernetes] Install Kafka Kraft Cluster by Bitnami Helm Chart'
tags:
  - Kubernetes
  - Kafka
categories:
  - DataEngineering
  - ApacheKafka
date: 2024-06-01T11:35:00+08:00
slug: k8s-bitnami-kafka-kraft-helm-chart
---

## TL; DR
This article records the use of bitnami helm chart to install kafka Kraft mode external cluster.

<!--more-->

## Prepare `values.yaml`

```yaml
global:
  storageClass: ceph-csi-rbd-hdd 

heapOpts: "-Xmx6g -Xms6g"

listeners:
  interbroker:
    name: INTERNAL
    containerPort: 9092
    protocol: PLAINTEXT
  controller:
    name: CONTROLLER
    containerPort: 9093
    protocol: PLAINTEXT
  client:
    name: CLIENT
    containerPort: 9095
    protocol: PLAINTEXT
  external:
    containerPort: 9094
    protocol: PLAINTEXT
    name: EXTERNAL

controller:
  replicaCount: 3
  persistence:
    size: 50Gi

broker:
  replicaCount: 3
  persistence:
    size: 300Gi

externalAccess:
  enabled: true
  controller:
    forceExpose: false
    service:
      type: NodePort
      ports:
        external: 9094
      nodePorts:
        - 30494
        - 30594
        - 30694
      useHostIPs: true
  broker:
    service:
      type: NodePort
      ports:
        external: 9094
      nodePorts:
        - 30194
        - 30294
        - 30394
      useHostIPs: true

volumePermissions:
  enabled: true

rbac:
  create: true

kraft:
  clusterId: M2VhY2Q3NGQ0NGYzNDg2YW
```

## Deploy

```yaml
helm upgrade --install -name kafka bitnami/kafka --namespace kafka -f values.yaml
```

![](kafka.png)

```yaml
Kafka can be accessed by consumers via port 9095 on the following DNS name from within your cluster:

    kafka.kafka.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9095 on the following DNS name(s) from within your cluster:

    kafka-controller-0.kafka-controller-headless.kafka.svc.cluster.local:9095
    kafka-controller-1.kafka-controller-headless.kafka.svc.cluster.local:9095
    kafka-controller-2.kafka-controller-headless.kafka.svc.cluster.local:9095
    kafka-broker-0.kafka-broker-headless.kafka.svc.cluster.local:9095
    kafka-broker-1.kafka-broker-headless.kafka.svc.cluster.local:9095
    kafka-broker-2.kafka-broker-headless.kafka.svc.cluster.local:9095
```

## Set Kafka Load Balance
You can see that both broker and controller are exposed its' service via NodePort after the deployment. And we can now set load balance for Kafka to provide the unified entrypoint to external client

![](kafka-svc.png)

Currently, an Nginx has been established for the cluster outside Kubernetes. We can use this server directly to also set up load forwarding for Kafka's TCP traffic. The configuration file is as follows. All Worker Node nodes that will be opened to the Node Port are set up at once:

```toml
upstream tcp9094 {
    server 172.20.37.36:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.36:30294 max_fails=3 fail_timeout=30s;  
    server 172.20.37.36:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.36:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.36:30594 max_fails=3 fail_timeout=30s;  
    server 172.20.37.36:30694 max_fails=3 fail_timeout=30s;
 
    server 172.20.37.37:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.37:30294 max_fails=3 fail_timeout=30s;
    server 172.20.37.37:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.37:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.37:30594 max_fails=3 fail_timeout=30s;
    server 172.20.37.37:30694 max_fails=3 fail_timeout=30s;

    server 172.20.37.38:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.38:30294 max_fails=3 fail_timeout=30s;
    server 172.20.37.38:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.38:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.38:30594 max_fails=3 fail_timeout=30s;
    server 172.20.37.38:30694 max_fails=3 fail_timeout=30s;

    server 172.20.37.39:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.39:30294 max_fails=3 fail_timeout=30s;
    server 172.20.37.39:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.39:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.39:30594 max_fails=3 fail_timeout=30s;
    server 172.20.37.39:30694 max_fails=3 fail_timeout=30s;

    server 172.20.37.40:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.40:30294 max_fails=3 fail_timeout=30s;
    server 172.20.37.40:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.40:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.40:30594 max_fails=3 fail_timeout=30s;
    server 172.20.37.40:30694 max_fails=3 fail_timeout=30s;
 
    server 172.20.37.41:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.41:30294 max_fails=3 fail_timeout=30s;
    server 172.20.37.41:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.41:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.41:30594 max_fails=3 fail_timeout=30s;
    server 172.20.37.41:30694 max_fails=3 fail_timeout=30s;  

    server 172.20.37.42:30194 max_fails=3 fail_timeout=30s;
    server 172.20.37.42:30294 max_fails=3 fail_timeout=30s;
    server 172.20.37.42:30394 max_fails=3 fail_timeout=30s;
    server 172.20.37.42:30494 max_fails=3 fail_timeout=30s;
    server 172.20.37.42:30594 max_fails=3 fail_timeout=30s;
    server 172.20.37.42:30694 max_fails=3 fail_timeout=30s;
}

server {
    listen        9094;
    proxy_pass    tcp9094;
 
    proxy_connect_timeout 300s;
    proxy_timeout 300s;
}
```

## Deploy Kafka UI

Prepare `values.yaml` file:
```yaml
yamlApplicationConfig:
  kafka:
    clusters:
      - name: platform
        bootstrapServers:  kafka-broker-headless:9092
  auth:
    type: disabled
  management:
    health:
      ldap:
        enabled: false
```
And then deploy the UI through helm chart.
```yaml
helm install -name kafka-ui kafka-ui/kafka-ui -f values.yaml --namespace kafka
```

![](ui.png)