---
title: "Apache Streampark Docker Compose Setup"
tags:
  - StreamPark
  - Flink
  - Docker
categories:
  - DataEngineering
  - ApacheFlink
date: 2023-12-09T10:58:11+08:00
slug: data-apache-streampark-docker-compose-setup
---

```yaml
services:
  docker:
    image: docker:24-dind
    privileged: true
    container_name: docker
    networks:
      - streampark
    command:
      - dockerd
      - --host=unix:///var/run/docker.sock
      - --host=tcp://0.0.0.0:2375
    restart: always
    healthcheck:
      test: ["CMD", "docker", "info"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 20s

  mysql:
    image: mysql:8.3
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "streampark"
      MYSQL_USER: "streampark"
      MYSQL_PASSWORD: "streampark"
    volumes:
      - dbdata:/var/lib/mysql
      - ./config/init:/docker-entrypoint-initdb.d
    user: "1000:1000"
    restart: always
    networks:
      - streampark
    healthcheck:
      test: >
        bash -c "mysql -uroot -pstreampark -Dstreampark -e 'SELECT COUNT(*) FROM t_user WHERE user_id=100000;' | grep -E -q '^[[:space:]]*1[[:space:]]*$'"
      interval: 10s
      timeout: 5s
      retries: 45
      start_period: 60s

  streampark:
    image: harbor.sdsp-stg.com/base/streampark-flink:2.1.4
    container_name: streampark
    command: ["bash", "-c", "bash ./bin/streampark.sh start_docker"]
    ports:
      - "10000:10000"
      - "10030:10030"
    environment:
      - TZ=Asia/Taipei
      - SPRING_PROFILES_ACTIVE=mysql
      - DOCKER_HOST=tcp://docker:2375
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/streampark?useSSL=false&useUnicode=true&characterEncoding=UTF-8&allowPublicKeyRetrieval=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&serverTimezone=GMT%2B8
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=streampark
    volumes:
      - ./config/kube:/root/.kube
      - data:/opt
      - ./config/application.yml:/streampark/conf/application.yml
      - ./config/application-mysql.yml:/streampark/conf/application-mysql.yml
    privileged: true
    restart: always
    networks:
      - streampark
    depends_on:
      docker:
        condition: service_healthy
      mysql:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:10000"]
      interval: 5s
      timeout: 5s
      retries: 12

  prometheus:
    image: prom/prometheus:v2.45.2
    container_name: prometheus
    restart: always
    networks:
      - streampark
    volumes:
      - ./config/prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:10.2.3
    container_name: grafana
    restart: always
    networks:
      - streampark
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: "!QAZxsw2"
    depends_on:
      - prometheus
    ports:
      - "3000:3000"

  pushgateway:
    image: prom/pushgateway:v1.10.0
    container_name: pushgateway
    networks:
      - streampark
    restart: always
    ports:
      - "9091:9091"

  node-exporter:
    image: prom/node-exporter:v1.6.1
    container_name: node-exporter
    networks:
      - streampark
    restart: always
    ports:
      - "9100:9100"

  minio:
    image: docker.io/bitnami/minio:2022
    restart: always
    container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    networks:
      - streampark
    volumes:
      - minio_data:/data
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=!QAZxsw2
      - MINIO_DEFAULT_BUCKETS=poc

  mssql:
    image: mcr.microsoft.com/mssql/server:2019-latest
    user: root
    restart: always
    container_name: mssql
    networks:
      - streampark
    ports:
      - "1433:1433"
    volumes:
      - mssql_data:/var/opt/mssql
    environment:
      - ACCEPT_EULA=Y
      - MSSQL_SA_PASSWORD=P@ssword*123456
      - MSSQL_PID=Developer

volumes:
  dbdata:
  data:
  prometheus_data:
  grafana_data:
  mssql_data:
  minio_data:

networks:
  streampark:
    driver: bridge
```
