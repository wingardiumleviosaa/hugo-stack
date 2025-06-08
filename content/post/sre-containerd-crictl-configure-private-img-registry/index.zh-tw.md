---
title: Containerd (crictl) 配置私有鏡像倉庫
tags:
  - Docker
categories:
  - SRE
date: 2022-03-16 18:23:00
slug: containerd-configure-private-harbor
---

前幾天將 k8s 升級到 1.23 版，使用 containerd 當 CRI，立馬就遇到要拉取私有 image registry 的狀況，本文紀錄配置過程。

<!--more-->

## 配置 containerd config.toml

依據遠端私有倉庫是否加密，有不同配置：

### - 私有倉庫帶有 tls 加密 (https)

```
vim /etc/containerd/config.toml
```

在 `[plugins."io.containerd.grpc.v1.cri".registry.mirror]` 下加上私有倉庫位址/URL

```
[plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.nexmasa.com"]
   endpoint = ["https://harbor.nexmasa.com"]
```

另外再接著加上 `[plugins."io.containerd.grpc.v1.cri".registry.configs]` 的項目

```
[plugins."io.containerd.grpc.v1.cri".registry.configs]
   [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.nexmasa.com".tls]
      ca_file   = "/etc/containerd/harbor.nexmasa.com/ca.pem"
      cert_file = "/etc/containerd/harbor.nexmasa.com/cert.pem"
      key_file  = "/etc/containerd/harbor.nexmasa.com/key.pem"
   [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.nexmasa.com".auth]
      username = "xxxxxx"
      password = 'xxxxxx'
```

</br>

![](https://imgur.com/86AZK4p.png)

### - 私有倉庫無加密 (http) 或是想略過憑證檢查

在 tls 配置項下，以 `insecure_skip_verify = true` 參數取代上面配置的金鑰及憑證。

```
[plugins."io.containerd.grpc.v1.cri".registry.configs]
   [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.nexmasa.com".tls]
      insecure_skip_verify = true
```

## 重啟 containerd

```
systemctl restart containerd
```

查看配置

```yaml
[root@node ~]# crictl info
{
  "status": {
    "conditions": [
      {
        "type": "RuntimeReady",
        "status": true,
        "reason": "",
        "message": ""
      },
      {
        "type": "NetworkReady",
        "status": true,
        "reason": "",
        "message": ""
      }
    ]
  },
.
.
.
"config": {
.
.
.
    "registry": {
      "mirrors": {
        "docker.io": {
          "endpoint": [
            "https://registry-1.docker.io"
          ]
        },
        "harbor.nexmasa.com": {
          "endpoint": [
            "https://harbor.nexmasa.com"
          ]
        }
      },
      "configs": {
        "harbor.nexmasa.com": {
          "auth": {
            "username": "xxxxx",
            "password": "xxxxx",
            "auth": "",
            "identitytoken": ""
          },
          "tls": {
            "insecure_skip_verify": false,
            "caFile": "/etc/containerd/harbor.nexmasa.com/ca.pem",
            "certFile": "/etc/containerd/harbor.nexmasa.com/cert.pem",
            "keyFile": "/etc/containerd/harbor.nexmasa.com/key.pem"
          }
        }
      },
      "auths": null,
      "headers": null
    },
```

## 測試

拉取鏡像

```
crictl pull harbor.nexmasa.com/test/hello-world@sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042
```

</br>

![](https://imgur.com/CPIvAP8.png)

## Reference

- https://github.com/containerd/containerd/blob/main/docs/cri/registry.md
