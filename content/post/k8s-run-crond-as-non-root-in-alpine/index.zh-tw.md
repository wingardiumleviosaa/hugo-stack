---
title: 'Run Crond as Non Root in Alpine Container by Pod/Deployment'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-10-23 19:04:00
slug: run-crond-as-non-root-in-alpine-container-by-pod-or-deployment
---
## TL;DR
在[上一篇文章](https://ulahsieh.netlify.app/p/docker-run-cronjob-as-non-root-user-in-alpine-container/)中試了在 alpine docker container 中使用 non root user 跑 crond，但將 build 好的 docker image 搬到 kubernetes 給 deployment 的 pod 使用時，卻會出現 initgroup operation not permitted 的錯誤。

<!--more-->

![](https://imgur.com/DGBzDOx.png)

踩坑踩了整整三天，該改的權限都改了，最後終於找到 supercronic 這個酷東西 T_T

## Dockerfile
這邊的 dockerfile 直接先下載好 supercronic build 好的 binary，再 COPY 進 image 中，也可以參考 [installation instruction](https://github.com/aptible/supercronic/releases) 在 build 的階段下載。
```docker
# We want to populate the module cache based on the go.{mod,sum} files.
COPY go.mod .
COPY go.sum .

RUN go mod download

COPY . .

# Build the Go app
RUN go build -o ./out/ccoe-bot .

# Start fresh from a smaller image
FROM harbor.wistron.com/base_image/alpine:3.12

USER root

RUN apk update && addgroup --gid 1000 ccoebot && adduser --disabled-password --ingroup ccoebot --uid "1000" ccoebot

COPY --from=build_base --chown=ccoebot:ccoebot /tmp/ccoe-bot/out/ccoe-bot /home/ccoebot/app/ccoe-bot
COPY bin/git-sync /usr/bin
COPY bin/terragrunt /usr/bin
COPY bin/terrascan /usr/bin
COPY bin/supercronic /usr/bin
COPY --chown=ccoebot:ccoebot init.sh /home/ccoebot/app

USER ccoebot
RUN git clone https://gitlab.wistron.com/ccoe/terrascan_policy.git /home/ccoebot/terrascan_policy
WORKDIR /home/ccoebot/terrascan_policy
RUN terrascan init -c terrascan-config.toml
WORKDIR /home/ccoebot/.terrascan
RUN git config --bool branch.master.sync true && git branch -D HEAD

# set scheduler for git-sync
RUN echo "*/10 * * * * cd /home/ccoebot/.terrascan && date >> /home/ccoebot/app/sync.log && /usr/bin/git-sync >> /home/ccoebot/app/sync.log" >> /home/ccoebot/app/mycron

# This container exposes port 8080 to the outside world
EXPOSE 8080

# Run the binary program produced by `go install`
#CMD ["/app/ccoe-bot"]
# repack the ccoe-bot and crond to init.sh

CMD ["/home/ccoebot/app/init.sh"]
```

## init.sh
這裡又是另一個要注意的地方，因為 supercronic 也是一個要跑的使用者程序，故這個案例同時會有兩個程序需要在 CMD 裡面一同跑起，使用下面的寫法完成在同個 container 中跑兩個程序。
```bash
#!/bin/bash

/usr/bin/supercronic /home/ccoebot/app/mycron &
P1=$!
/home/ccoebot/app/ccoe-bot &
P2=$!
wait $P1 $P2
```
或是
```bash
#!/bin/bash

# Start the first process
/usr/bin/supercronic /home/ccoebot/app/mycron &
  
# Start the second process
/home/ccoebot/app/ccoe-bot &
  
# Wait for any process to exit
wait -n
  
# Exit with status of process that exited first
exit $?
```

## kubernetes deployment yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-ccoebot
  namespace: atlantis
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: dev-ccoebot
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    spec:
      containers:
        - image: harbor.wistron.com/k8sprdwhqccoe/ccoe-bot:nonroot
          imagePullPolicy: IfNotPresent
          name: dev-ccoebot
          resources:
            requests:
              memory: '256Mi'
              cpu: '512m'
            limits:
              memory: '1024Mi'
              cpu: '1024m'
          securityContext:
            runAsUser: 1000
            runAsGroup: 1000
            allowPrivilegeEscalation: false
            privileged: false
            readOnlyRootFilesystem: false
            runAsNonRoot: true
          stdin: true
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          tty: true
      dnsPolicy: ClusterFirst
      imagePullSecrets:
        - name: hbsrt
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        runAsNonRoot: true
      terminationGracePeriodSeconds: 30
```

## Result

![](https://imgur.com/gOsbjac.png)

## Reference
拯救我用 supercronic 的文章, 感謝 alesk 大
- https://gist.github.com/alesk/33b716f04cdce0751473b8232405dc32
Run Mutiple Process in Container:
- https://docs.docker.com/config/containers/multi-service_container/
- https://stackoverflow.com/a/56663151