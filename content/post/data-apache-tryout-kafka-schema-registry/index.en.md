---
title: "Kafka Schema Registry Installation and Usage"
tags:
  - Kafka
categories:
  - DataEngineering
  - ApacheKafka
date: 2024-11-22T21:31:00+08:00
slug: data-apache-tryout-kafka-schema-registry
---

## TL;DR

In a Kubernetes environment with an existing Kafka cluster deployed using Strimzi, this post documents how to deploy and use Kafka Schema Registry.

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cp-schema-registry
  namespace: kafka
  labels:
    app: cp-schema-registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cp-schema-registry
  template:
    metadata:
      labels:
        app: cp-schema-registry
    spec:
      containers:
        - name: cp-schema-registry-server
          image: "confluentinc/cp-schema-registry:7.7.1"
          imagePullPolicy: "IfNotPresent"
          ports:
            - name: schema-registry
              containerPort: 8081
              protocol: TCP
          env:
            - name: SCHEMA_REGISTRY_HOST_NAME
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
            - name: SCHEMA_REGISTRY_LISTENERS
              value: http://0.0.0.0:8081
            - name: SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS
              value: PLAINTEXT://my-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092

---
apiVersion: v1
kind: Service
metadata:
  name: cp-schema-registry
  namespace: kafka
  labels:
    app: cp-schema-registry
spec:
  ports:
    - name: schema-registry
      port: 8081
  selector:
    app: cp-schema-registry

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cp-schema-registry
  namespace: kafka
  labels:
    app: cp-schema-registry
spec:
  ingressClassName: nginx
  rules:
    - host: kafka-schema-registry.sdsp-dev.com
      http:
        paths:
          - backend:
              service:
                name: cp-schema-registry
                port:
                  name: schema-registry
            path: /
            pathType: Prefix
  tls:
    - hosts:
        - kafka-schema-registry.sdsp-dev.com
```

![](getKafka.png)

## Update Kafka UI Configuration

```yaml
yamlApplicationConfig:
  kafka:
    clusters:
      - name: dev
        bootstrapServers: my-cluster-kafka-bootstrap:9092
        schemaRegistry: http://cp-schema-registry:8081
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
  host: kafka-ui.sdsp-dev.com
  tls:
    enabled: true
```

Update the chart:

```yaml
helm upgrade kafka-ui kafka-ui/kafka-ui --namespace kafka -f values.yaml
```

## Test

From Kafka UI, access the Schema Registry and create a schema `test-value` with the following structure:

```yaml
{
  "type": "record",
  "name": "TestRecord",
  "fields":
    [{ "name": "id", "type": "int" }, { "name": "name", "type": "string" }],
}
```

![](addSchema.png)

Use the Python client to produce data using the specified schema:

```python
# main.py
import requests
import json
from confluent_kafka import SerializingProducer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer
from confluent_kafka.schema_registry import Schema

kafka_url = 'kafka.sdsp-dev.com:8094'
schema_registry_url = 'https://kafka-schema-registry.sdsp-dev.com/'
kafka_topic = 'test'
schema_registry_subject = 'test-value'

def delivery_report(errmsg, msg):
    """Delivery report callback."""
    if errmsg is not None:
        print("Delivery failed for Message: {} : {}".format(msg.key(), errmsg))
        return
    print('Message: {} successfully produced to Topic: {} Partition: [{}] at offset {}'.format(msg.key(), msg.topic(), msg.partition(), msg.offset()))

def avro_producer(kafka_url, schema_registry_url, schema_registry_subject):
    # Get schema from Schema Registry
    sr, latest_version = get_schema_from_schema_registry(schema_registry_url, schema_registry_subject)
    print(latest_version.schema.schema_str)

    value_avro_serializer = AvroSerializer(
        schema_registry_client=sr,
        schema_str=latest_version.schema.schema_str,
        conf={
            'auto.register.schemas': False
        }
    )

    # Kafka Producer
    producer = SerializingProducer({
        'bootstrap.servers': kafka_url,
        'security.protocol': 'plaintext',
        'value.serializer': value_avro_serializer,
        'delivery.timeout.ms': 120000,
        'enable.idempotence': 'true'
    })

    # Test data
    test_data = [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"},
        {"id": 3, "name": "Charlie"}
    ]

    # Produce
    for record in test_data:
        try:
            producer.produce(topic=kafka_topic, value=record, on_delivery=delivery_report)
            producer.poll(1)

        except Exception as e:
            print(f"Failed to produce record: {record}, Error: {e}")

    producer.flush()

def get_schema_from_schema_registry(schema_registry_url, schema_registry_subject):
    """Get schema from Schema Registry"""
    sr = SchemaRegistryClient({'url': schema_registry_url})
    latest_version = sr.get_latest_version(schema_registry_subject)
    return sr, latest_version

avro_producer(kafka_url, schema_registry_url, schema_registry_subject)
```

```bash
python main.py
```

View the result:

![](result.png)
