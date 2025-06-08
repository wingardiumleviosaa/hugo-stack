---
title: "[Docker] docker build network error"
tags:
  - Docker
categories:
  - DevOps
date: 2021-08-31 21:09:00
slug: docker-build-network-error
---

在 build docker image 時發生 `network error` 的錯誤

<!--more-->

## Dockerfile

```docker
FROM golang:1.15.3-alpine3.12 AS builder
WORKDIR /
COPY . .

RUN apk update && apk add --update git
RUN CGO_ENABLED=0 go build -installsuffix cgo -o /it-preprocess-adapter ./cmd/it-preprocess-adapter/it-preprocess-adapter.go

FROM alpine:3.12
COPY --from=builder /it-preprocess-adapter /it-preprocess-adapter
COPY ./configs /configs
COPY ./settings /settings
COPY ./build/docker/startup.sh /startup.sh

RUN chmod 2777 -R /settings
#USER 1001

CMD ["sh", "/startup.sh"]
```

## Docker build error

![](https://imgur.com/QCOL9ZO.png)

## Solution

重啟 docker 就解決了

```bash
systemctl restart docker
```

## Reference

https://github.com/laradock/laradock/issues/2551
