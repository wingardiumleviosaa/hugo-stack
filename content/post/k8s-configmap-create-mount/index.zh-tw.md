---
title: 'ConfigMap 建立及掛載'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2021-01-08 14:26:00
slug: kubernets-configmap
---

## ConfigMap
ConfigMap 以 key-vaule 的方式用來描述系統相關設定，所有與應用程式相關的**非敏感性**未加密的資訊可放在 ConfigMap 內。而如有敏感性資料，則需透過 `Secret`。

<!--more-->

### 主要目的
主要目的是將應用程式與設定解耦，ConfigMap 與 Pod 將個別單獨存在於 k8s 叢集中，當 Pod 需要使用 ConfigMap 時才需要將 ConfigMap 掛載到 Pod 內使用。解耦的好處有：
- 便於管理
- 彈性高，可掛載不同的 ConfigMap 到 Pod 內使用；或是同一個 ConfigMap 掛載到多個 Pod。

### 用法
Kubernetes 的 ConfigMap 透過 kubectl create 或 kubectl apply 來建立。
```
$ kubectl create configmap [資源名字] [來源參數]
$ kubectl apply condfigmap.yaml
```
#### 使用 kubectl create 建立
使用 kubectl create 可以從檔案路徑、檔案或是 literal value 來建立 configMap。
- --from-file
```sh=
#  建立名為 myConf、資料來源是某路徑下所有檔案的 configMap
$ kubectl create configmap myConf --from-file=/path/for/config/file/

#  建立名為 myConf、資料來源是一個檔案的 configMap
$ kubectl create configmap myConf --from-file=/path/to/app.properties

#  建立名為 myConf、資料來源是多個檔案的 configMap
$ kubectl create configmap myConf --from-file=/path/of/app1.properties --from-file=/path/of/app2.properties
```

</br>

{{< notice note >}}
如果是來源是檔案的話，則 configMap 中的 key 就會是檔名，value 則是檔案內容。
{{< /notice >}}

- --from-literal
```
#  建立名為 myConf、包含指定鍵值對的 configMap
$ kubectl create configmap myConf --from-literal=key1=config1

$ kubectl create configmap myConf --from-literal=key1=config1 --from-literal=key2=config2
```

- 兩個共用
```
$ kubectl create configmap myConf --from-file=/path/of/config.conf \
    --from-literal=key1=config1 \
    --from-literal=key2=config2
```

- --from-env-file
使用環境變數表示的檔案。

{{< notice warning >}}
請注意，value 如果有 "" 則會視為是值的一部份。且如果在同個 create 中使用多個 --from-env-file 則指會應用最後一個。
{{< /notice >}}

#### 使用 kubectl apply yaml 檔案建立
準備 yaml 檔
```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myConf
data:
  key1: config1
  key2: config2
  app.properties: | 
    property.1 = value1 
    property.2 = value2 
    property.3 = value3
```
佈署
```
$ kubectl apply -f configmap.yaml
```

### 查看 ConfigMap
建立完成後可以使用 `kubectl get` 或是 `kubectl describe` 的擷取 configMap 的內容。

```
$ kubectl get configmaps myConf -o yaml
```

</br>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: 2021-01-04T18:52:05Z
  name: myConf
  namespace: default
  resourceVersion: "516"
  uid: b4952dc3-d670-11e5-8cd0-68f728db1985
data:
  key1: config1
  key2: config2
  app.properties: | 
    property.1 = value1 
    property.2 = value2 
    property.3 = value3
```

### 將 ConfigMap 掛載到 pod 使用

#### 當成環境變數使用

pod 的 yaml 檔如下
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: testenv
spec:
  containers:
    - name: test
      image: tomcat:8
      imagePullPolicy: IfNotPresent
      command: [ "/bin/sh", "-c", "echo $(KEY1_ENV)" ]
      env:
        - name: KEY1_ENV
          valueFrom:
            configMapKeyRef:
              name: myConf
              key: key1
```
將 pod 跑起來後，ConfigMap myConf 中的 key1 的 value 就會做為環境變數 KEY1_ENV 的值。

#### 掛載成 volume

pod 的 yaml 檔如下
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: testvolume
spec:
  containers:
    - name: test
      image: tomcat: 8
      imagePullPolicy: IfNotPresent
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: myConf
```

</br>

{{< notice note >}}
使用 volume 將 ConfigMap 作為文件或目錄直接掛載，ConfigMap 中每一個 key-value 鍵值對都會生成一個文件，key 為文件名，value 為內容。
{{< /notice >}}

另一種方式，只掛載某個 key，並指定相對路徑。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: testvolume
spec:
  containers:
    - name: test
      image: tomcat:8
      imagePullPolicy: IfNotPresent
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
         name: myConf
         items:
           - key: key1
             path: /path/to/key1  # key1 會放在 mountPath /etc/config/path/to 下。
           - key: app.properties
             path: app.properties # 如果 path 與 key 相同，則會直接把 app.properties 文件放在 mountPath 下。
```


### Reference
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
- https://www.cnblogs.com/pu20065226/p/10690628.html