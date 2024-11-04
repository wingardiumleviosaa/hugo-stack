---
title: Istio 沒掛，但正確的設置 gateway 跟 virtual service 後，卻一直 404 not found
tags:
  - Istio
categories:
  - Kubernetes
date: 2022-03-24 21:52:00
slug: istio-404-not-found
---
手上有一個寫好的 API 要對外釋出，在設置完 istio 資源之後，curl istio ingress gateway/targetAPI 卻一直回傳 404 Not Found，照理來說這個 API 如果找不到資料回傳的 404 訊息應該是 `{"error":"Record Not Found, the serial number doesn't exist"}`，用這篇文章記錄問題跟解決方式。

<!--more-->

## 問題排查
1. 在相同的 istio ingress gateway 上的 API 皆正常運作，排除 istio 本身可能會有問題
2. 用同樣的配置檔，部屬在另外一個 K8s 環境上的 istio，發現運作正常，排除配置檔有誤的問題
3. 用其他 API 部屬，也一樣直接 404 Not Found，排除原先 API 本身可能有誤的問題
4. 查看 istio ingress gateway 的 log
```
[2022-03-24T06:23:07.653Z] "GET /api/convergence/findRecord/TBCC32008806 HTTP/1.1" 200 - "-" "-" 0 9491 5 4 "10.1.5.32" "PostmanRuntime/7.29.0" "3655dd39-9c2c-9c14-ac5c-53fc7547a155" "10.1.5.41" "10.244.64.149:8080" outbound|8080||converg-api.converg-api.svc.cluster.local 10.244.128.48:42104 10.244.128.48:8080 10.1.5.32:39159 - http-Sbups
---
[2022-03-24T06:26:32.759Z] "GET /api/convergence/grabreflow/TBCBB2039913 HTTP/1.1" 404 - "-" "-" 0 18 1 1 "10.1.5.32" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:98.0) Gecko/20100101 Firefox/98.0" "61998a8c-0c79-94a0-9c84-cb03a2f10390" "10.1.5.41" "10.244.128.19:8080" outbound|8080||reverseapi.converg-apipost.svc.cluster.local 10.244.128.48:36740 10.244.128.48:8080 10.1.5.32:15717 - http-XbhqV
```
上面 200 的是正常運作的 API，下面 404 是新設置的 API，所以其實 404 這個 istio gw & virtual service 其實是有運作的，得出來的結論就是，<font style="background:PeachPuff"><u>istio 找不到新設置的 API 去路由，極大可能是跟其他 URL 規則衝突。</u></font>

## 解決
終於發現前面最後一個設置的 API 沒有設 prefix，所以 istio 就直接監聽 `/`，導致後面怎麼設新的 API，都認不到!!!!!!!!
```yaml
kubectl get virtualservices.networking.istio.io -n converg-apipost converg-apipost-vs -o yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  creationTimestamp: "2021-08-04T10:04:00Z"
  generation: 9
  labels:
    protocol: http
  name: converg-apipost-vs
  namespace: converg-apipost
  resourceVersion: "241532937"
  selfLink: /apis/networking.istio.io/v1beta1/namespaces/converg-apipost/virtualservices/converg-apipost-vs
  uid: 15049a96-f97f-4b00-bee9-001fafcdd2ff
spec:
  gateways:
  - converg-apipost-gw
  hosts:
  - '*'
  http:
  - match:
    - uri:
        prefix: ""
    name: http-OuFqP
    route:
    - destination:
        host: reverseapi
        port:
          number: 8080
```
把 prefix 加上去後，原本後加的 API 就成功運作了 :v:
```json
[root@k8sm1 ~]# curl 10.1.5.41/api/convergence/grabreflow/test
{"error":"Record Not Found, the serial number doesn't exist"}
```