---
title: >-
  Kustomize Can’t Render the ConfigMap Hashing Name to CronJob
  Resource
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-09-06 21:02:00
slug: kustomization-render-failed-in-cronjob
---
## 問題
在使用 kustomize 配置 Kubernetes 資源時，Kustomization 定義的 ConfigMap 無法正確的渲染到 CronJob 資源中。
原 yaml 檔如下:

<!--more-->

```yaml
# 建立 templatevar 文件
cat <<EOF >templatevar
FOO=Bar
EOF

# 建立 cronjob 文件
cat <<EOF >cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
  namespace: infrase
spec:
  schedule: "25,45,05 * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            resources:
              limits:
                cpu: "1"
                memory: 500Mi
            imagePullPolicy: IfNotPresent
            securityContext:
              runAsNonRoot: true
              runAsUser: 1000
              allowPrivilegeEscalation: false
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster && cat /config/templatevar
            volumeMounts:
            - mountPath: /config/
              name: templatevar
          volumes:
          - name: templatevar
            configMap:
              name: templatevar
          restartPolicy: OnFailure
EOF

# 建立 kustomization 文件
cat <<EOF >./kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - cronjob.yaml

configMapGenerator:
- name: templatevar
  files:
  - templatevar
EOF
```

使用 kustomize 渲染後，可以看到 CronJob 中指定的 ConfigMap 沒有正確吃到 configMapGenerator 所產生的檔案。

```
kubectl kustomize ./
```

</br>

```yaml
apiVersion: v1
data:
  templatevar: "FOO=Bar"
kind: ConfigMap
metadata:
  name: templatevar-tk9cdghbt6
  namespace: infrase
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
  namespace: infrase
spec:
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster && cat /config/templatevar
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            name: hello
            securityContext:
              allowPrivilegeEscalation: false
              runAsNonRoot: true
              runAsUser: 1000
            volumeMounts:
            - mountPath: /config/
              name: templatevar
          imagePullSecrets:
          - name: hbsrt
          restartPolicy: OnFailure
          volumes:
          - configMap:
              name: templatevar
            name: templatevar
  schedule: 57 * * * *
```

## 解決方式
需要在 kustomiztion 檔案中指定 namespace
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: infrase
resources:
  - cronjob.yaml
  
configMapGenerator:
- name: templatevar
  files:
  - templatevar
```

## Reference
- https://github.com/kubernetes-sigs/kustomize/issues/1301