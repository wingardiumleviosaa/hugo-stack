---
title: "[kong] Single Gateway and Konga Deployment by Docker Compose"
tags:
  - Kong
categories:
  - SRE
date: 2024-03-23 07:34:00
slug: single-kong-gw-deploy-by-docker-compose
---

### TL; DR

Kong 有不同部署架構，本文僅示範單一的 kong gateway 節點使用 Docker Compose 的安裝方式，包含 kong Gateway、PostgreSQL 以及第三方開源 Kong GUI Konga。

<!--more-->

{{< notice note >}}
Konga 是一個非官方的第三方開源的 kong 圖形化管理工具，使用 Node.js 開發，不過隨著 Kong Gateway 版本的更新，許多三方開源管理面板都缺乏及時維護而無法使用了。之後建議直接使用 Kong Gateway OSS 在 2023 開源的官方管理界面 Kong Manager。
{{< /notice >}}

### Docker Compose YAML

```yaml
version: "3.7"

volumes:
  kong_data: {}

networks:
  kong-net:

services:
  #######################################
  # Postgres: The database used by Kong
  #######################################
  kong-database:
    image: postgres:10
    container_name: kong-postgres
    restart: on-failure
    networks:
      - kong-net
    volumes:
      - kong_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: kong
      POSTGRES_PASSWORD: ${KONG_PG_PASSWORD:-kong}
      POSTGRES_DB: kong
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "kong"]
      interval: 30s
      timeout: 30s
      retries: 3

  #######################################
  # Kong database migration
  #######################################
  kong-migration:
    image: ${KONG_DOCKER_TAG:-kong:latest}
    command: kong migrations bootstrap
    networks:
      - kong-net
    restart: on-failure
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_DATABASE: kong
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: ${KONG_PG_PASSWORD:-kong}
    depends_on:
      - kong-database

  #######################################
  # Kong: The API Gateway
  #######################################
  kong:
    image: ${KONG_DOCKER_TAG:-kong:latest}
    restart: on-failure
    networks:
      - kong-net
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_DATABASE: kong
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: ${KONG_PG_PASSWORD:-kong}
      KONG_PROXY_LISTEN: 0.0.0.0:8000, 0.0.0.0:8443 ssl
      # KONG_PROXY_LISTEN_SSL: 0.0.0.0:8443
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    depends_on:
      - kong-database
    healthcheck:
      test: ["CMD", "kong", "health"]
      interval: 10s
      timeout: 10s
      retries: 10
    ports:
      - "8000:8000"
      - "8001:8001"
      - "8443:8443"
      - "8444:8444"

  #######################################
  # Konga database prepare
  #######################################
  konga-prepare:
    image: pantsel/konga:latest
    command: "-c prepare -a postgres -u postgresql://kong:${KONG_PG_PASSWORD:-kong}@kong-database:5432/konga"
    networks:
      - kong-net
    restart: on-failure
    depends_on:
      - kong-database

  #######################################
  # Konga: Kong GUI
  #######################################
  konga:
    image: pantsel/konga:latest
    restart: always
    networks:
      - kong-net
    environment:
      DB_ADAPTER: postgres
      DB_URI: postgresql://kong:${KONG_PG_PASSWORD:-kong}@kong-database:5432/konga
      NODE_ENV: production
    depends_on:
      - kong-database
    ports:
      - "1337:1337"
```
