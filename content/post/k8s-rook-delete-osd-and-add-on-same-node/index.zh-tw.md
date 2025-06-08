---
title: "[Kubernetes] Rook 刪除原先 /dev/sdb 的 OSD 並重新新增同個節點的 /dev/sda 的 OSD"
tags:
  - Kubernetes
  - Ceph
categories:
  - Kubernetes
date: 2024-04-30T14:45:00+08:00
slug: k8s-rook-delete-osd-and-add-on-same-node
---

## TL; DR

原本使用 Kubernetes 的三個 worker node 節點建立了擁有三個 node 的 rook，但因為資源不足的緣故，將其中一個節點由原本的 VM 改成以實機的方式加入集群，導致硬碟原本是以 /dev/sdb 的方式加入 OSD，但現在需要改成 /dev/sda。

<!--more-->

1. 先將 rook-ceph-operator 停用

```yaml
kubectl -n rook-ceph scale deployment rook-ceph-operator --replicas=0
```

2. 修改 rook cluster 配置，指定節點使用 sda 硬碟。

```yaml
# kubectl edit cephclusters.ceph.rook.io -n rook-ceph rook-ceph
storage:
  flappingRestartIntervalHours: 0
  nodes:
    - devices:
        - name: sda
      name: node5
  store: {}
  useAllDevices: true
  useAllNodes: true
```

3. 部署 ceph toolbox

```yaml
kubectl apply -f toolbox.yaml
```

```yaml
# toolbox.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rook-ceph-tools
  namespace: rook-ceph # namespace:cluster
  labels:
    app: rook-ceph-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rook-ceph-tools
  template:
    metadata:
      labels:
        app: rook-ceph-tools
    spec:
      dnsPolicy: ClusterFirstWithHostNet
      serviceAccountName: rook-ceph-default
      containers:
        - name: rook-ceph-tools
          image: quay.io/ceph/ceph:v18.2.2
          command:
            - /bin/bash
            - -c
            - |
              # Replicate the script from toolbox.sh inline so the ceph image
              # can be run directly, instead of requiring the rook toolbox
              CEPH_CONFIG="/etc/ceph/ceph.conf"
              MON_CONFIG="/etc/rook/mon-endpoints"
              KEYRING_FILE="/etc/ceph/keyring"

              # create a ceph config file in its default location so ceph/rados tools can be used
              # without specifying any arguments
              write_endpoints() {
                endpoints=$(cat ${MON_CONFIG})

                # filter out the mon names
                # external cluster can have numbers or hyphens in mon names, handling them in regex
                # shellcheck disable=SC2001
                mon_endpoints=$(echo "${endpoints}"| sed 's/[a-z0-9_-]\+=//g')

                DATE=$(date)
                echo "$DATE writing mon endpoints to ${CEPH_CONFIG}: ${endpoints}"
                  cat <<EOF > ${CEPH_CONFIG}
              [global]
              mon_host = ${mon_endpoints}

              [client.admin]
              keyring = ${KEYRING_FILE}
              EOF
              }

              # watch the endpoints config file and update if the mon endpoints ever change
              watch_endpoints() {
                # get the timestamp for the target of the soft link
                real_path=$(realpath ${MON_CONFIG})
                initial_time=$(stat -c %Z "${real_path}")
                while true; do
                  real_path=$(realpath ${MON_CONFIG})
                  latest_time=$(stat -c %Z "${real_path}")

                  if [[ "${latest_time}" != "${initial_time}" ]]; then
                    write_endpoints
                    initial_time=${latest_time}
                  fi

                  sleep 10
                done
              }

              # read the secret from an env var (for backward compatibility), or from the secret file
              ceph_secret=${ROOK_CEPH_SECRET}
              if [[ "$ceph_secret" == "" ]]; then
                ceph_secret=$(cat /var/lib/rook-ceph-mon/secret.keyring)
              fi

              # create the keyring file
              cat <<EOF > ${KEYRING_FILE}
              [${ROOK_CEPH_USERNAME}]
              key = ${ceph_secret}
              EOF

              # write the initial config file
              write_endpoints

              # continuously update the mon endpoints if they fail over
              watch_endpoints
          imagePullPolicy: IfNotPresent
          tty: true
          securityContext:
            runAsNonRoot: true
            runAsUser: 2016
            runAsGroup: 2016
            capabilities:
              drop: ["ALL"]
          env:
            - name: ROOK_CEPH_USERNAME
              valueFrom:
                secretKeyRef:
                  name: rook-ceph-mon
                  key: ceph-username
          volumeMounts:
            - mountPath: /etc/ceph
              name: ceph-config
            - name: mon-endpoint-volume
              mountPath: /etc/rook
            - name: ceph-admin-secret
              mountPath: /var/lib/rook-ceph-mon
              readOnly: true
      volumes:
        - name: ceph-admin-secret
          secret:
            secretName: rook-ceph-mon
            optional: false
            items:
              - key: ceph-secret
                path: secret.keyring
        - name: mon-endpoint-volume
          configMap:
            name: rook-ceph-mon-endpoints
            items:
              - key: data
                path: mon-endpoints
        - name: ceph-config
          emptyDir: {}
      tolerations:
        - key: "node.kubernetes.io/unreachable"
          operator: "Exists"
          effect: "NoExecute"
          tolerationSeconds: 5
```

4. 進入 toolbox 執行移除 sdb osd 操作

```yaml
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```

手動移除對應 osd

```yaml
ceph osd set noup
ceph osd down 0
ceph osd out 0
# 查看数据均衡进度， 等待数据均衡完成
ceph -w
# 均衡資料完成後移除對應的osd
ceph osd purge 0 --yes-i-really-mean-it
ceph auth del osd.0
ceph osd crush remove node5
```

檢查 ceph 狀態以及 osd 狀態

```
ceph -s
ceph osd tree
```

5. 移除 pod，並判斷刪除對應的 job

```
kubectl delete deploy -n rook-ceph rook-ceph-osd-0
```

6. 進入 node5 節點，並清除磁盤資料

```yaml
 #!/usr/bin/env bash
  DISK="/dev/sdb"
  # Zap the disk to a fresh, usable state (zap-all is important, b/c MBR has to be clean)
  # You will have to run this step for all disks.
  sgdisk --zap-all $DISK

  # These steps only have to be run once on each node
  # If rook sets up osds using ceph-volume, teardown leaves some devices mapped that lock the disks.
  ls /dev/mapper/ceph-* | xargs -I% -- dmsetup remove %
  # ceph-volume setup can leave ceph-<UUID> directories in /dev (unnecessary clutter)
  rm -rf /dev/ceph-*

  lsblk -f
  rm  -rf /var/lib/rook/*
```

7. 重新將 rook-ceph-operator 啟用，node5 sda osd 便會自動新增

```yaml
kubectl -n rook-ceph scale deployment rook-ceph-operator --replicas=0
```

## Reference

- https://blog.csdn.net/fancy106/article/details/121706998
