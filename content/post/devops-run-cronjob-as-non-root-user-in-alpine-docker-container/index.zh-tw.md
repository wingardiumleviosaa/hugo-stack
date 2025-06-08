---
title: "[Docker] Run Cronjob as Non Root User in Alpine Container"
tags:
  - Docker
categories:
  - DevOps
date: 2022-10-23 19:02:00
slug: docker-run-cronjob-as-non-root-user-in-alpine-container
---

## TL;DR

apline image 預設只能給 root 執行 crond，但剛好遇到有不允許 container 使用 root 執行的安全政策的情況。本篇記錄如何在 apline container 中使用 non root user 執行 crond。

<!--more-->

## Dockerfile

```docker
FROM harbor.wistron.com/base_image/alpine:3.12

USER root
RUN apk update && apk --no-cache add dcron libcap

RUN addgroup --gid 1000 ula && adduser --disabled-password --ingroup ula --uid "1000" ula

RUN chown ula:ula /usr/sbin/crond && setcap cap_setgid=ep /usr/sbin/crond

COPY --chown=ula:ula init.sh /home/ula/app
RUN touch /etc/crontabs/ula && chown -R ula:ula /etc/crontabs/ula

USER ula

# set scheduler
RUN echo "*/10 * * * * date >> /home/ccoebot/app/sync.log" >> /etc/crontabs/ula

# This container exposes port 8080 to the outside world
EXPOSE 8080

CMD ["/home/ula/app/init.sh"]
```

## init.sh

```sh
#!/bin/bash
crond -l 8
```

## Result

![](https://imgur.com/7AHSUXG.png)

## Reference

- https://github.com/gliderlabs/docker-alpine/issues/381
