---
title: Helm Migrate stable/nginx-ingress to ingress-nginx
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2022-05-22 21:17:00
slug: helm-migrate-stable-nginx-ingress-to-ingress-nginx
---

## 前言
原先集群使用的 helm chart 為 stable/nginx-ingress，而此 helm chart 已經被棄用，若 nginx 維持在舊版的話，之後新的漏洞修補都無法被含括。

此篇記錄如何將集群上面跑的 nginx-ingress-controller 換成新的版本的 chart ingress-nginx/ingress-nginx。

<!--more-->

## Current stable/nginx-ingress

原先 nginx-ingress 使用的版本

```yaml
$ kubectl exec -it -n nginx-ingress nginx-ingress-controller-585bb7f5b4-2nlzz -- /nginx-ingress-controller
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v0.34.1
  Build:         v20200715-ingress-nginx-2.11.0-8-gda5fa45e2
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.1
-------------------------------------------------------------------------------
```

查詢該 helm chart 有沒有更新的版本可直接更新，但結果如下，目前集群安裝的已經是該 chart 的最新版本了，且已標示 deprecated 不會再維護。

```bash
$ helm search repo stable/nginx-ingress --versions
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
stable/nginx-ingress    1.41.3          v0.34.1         DEPRECATED! An nginx Ingress controller that us...
stable/nginx-ingress    1.41.2          v0.34.1         An nginx Ingress controller that uses ConfigMap...
stable/nginx-ingress    1.41.1          v0.34.1         An nginx Ingress controller that uses ConfigMap...
stable/nginx-ingress    1.41.0          v0.34.0         An nginx Ingress controller that uses ConfigMap...
stable/nginx-ingress    1.40.3          0.32.0          An nginx Ingress controller that uses ConfigMap...
stable/nginx-ingress    1.40.2          0.32.0          An nginx Ingress controller that uses ConfigMap...
stable/nginx-ingress    1.40.1          0.32.0          An nginx Ingress
```

## Install ingress-nginx/ingress-nginx

安裝operator

```bash
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ kubectl create ns ingress-nginx
$ helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx
```

完成後應該會看到以下輸出

```bash
[root@master1 ~]# helm install ingress-nginx ingress-nginx/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
NAME: ingress-nginx
LAST DEPLOYED: Wed Aug 18 13:41:42 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

## 驗證安裝

```bash
$ kubectl exec -it -n ingress-nginx ingress-nginx-controller-b65df6fbb-jx4tf -- /nginx-ingress-controller --version
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v0.48.1
  Build:         30809c066cd027079cbb32dccc8a101d6fbffdcb
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.20.1

-------------------------------------------------------------------------------
```

## Create ingress resource

準備 ingress resource 的 yaml 檔，請注意雖然上方的安裝成功的訊息有示範 ingress 的 yaml，但因為 [networking.k8s.io/v1beta1](http://networking.k8s.io/v1beta1) 已在 Kubernetes 1.19+ 被棄用，如果維持使用，則會遇到 `Warning: [networking.k8s.io/v1beta1](http://networking.k8s.io/v1beta1) Ingress is deprecated in v1.19+, unavailable in v1.22+; use [networking.k8s.io/v1](http://networking.k8s.io/v1) Ingress`的錯誤，所以參考ingress-nginx([https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/](https://kubernetes.github.io/ingress-nginx/user-guide/basic-usage/)) 的官網，改成以下格式：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myservicea
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myservicea.foo.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myservicea
            port:
              number: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myserviceb
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: myserviceb.foo.org
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myserviceb
            port:
              number: 80
```

## 刪除舊的 ingress controller

確認所有流量都已經導到新的 controller 後，就可以把舊的 stable/nginx-ingress 的 controller 刪掉了。

```bash
$ helm uninstall nginx-ingress
```

## Reference
- [https://jfrog.com/blog/migrate-nginx-from-stable-helm-charts-with-chartcenter/](https://jfrog.com/blog/migrate-nginx-from-stable-helm-charts-with-chartcenter/)
- 補充：如果要實現 zero-downtime 的部屬，可以參考這篇文章 [https://medium.com/codecademy-engineering/kubernetes-nginx-and-zero-downtime-in-production-2c910c6a5ed8](https://medium.com/codecademy-engineering/kubernetes-nginx-and-zero-downtime-in-production-2c910c6a5ed8)