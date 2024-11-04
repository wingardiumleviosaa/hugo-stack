---
title: 為對外的 istio gateway 加上 https
tags:
  - Istio
categories:
  - Kubernetes
date: 2022-05-19 09:11:00
slug: add-https-to-istio-gw
---
本篇文章記錄怎麼使用 cert-manager 為對外的 istio gateway 加上 https。

## 憑證分類
1. **自簽憑證**：某些不需要被公開存取、但希望達到資料傳輸能加密的內部服務，可以使用自簽憑證，Client 去存取的時候自己帶上 CA 憑證去驗證即可，例如 HashiCorp Vault, AWS RDS TLS 連線...等。

2. **第三方 CA 機構簽發憑證**：如果是公開的網路服務，就必須透過正規的 CA 機構來簽發，如需要收費的 Digicert, SSL.com, Symantec...等，或是免費的 Let’s Encrypt。

<!--more-->

## cert-manager

![](https://imgur.com/L6mxIul.png)

cert-manager 是基於 Kubernetes 所開發的憑證管理工具，它可以可以幫忙發出來自各家的 TLS 憑證，例如上面所提到的 ACME (Let’s Encrypt), HashiCorp Vault, Venafi 或是自己簽發的憑證，而且它還可以確保 TLS 憑證一直維持在有效期限內。

[Above Reference](https://medium.com/starbugs/%E4%BD%BF%E7%94%A8-cert-manager-%E7%AE%A1%E7%90%86-k8s-tls-%E6%86%91%E8%AD%89-ab6258af9195)

## Install

```bash
$ kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.4.0/cert-manager.yaml
$ kubectl get pods --namespace cert-manager
  NAME                                       READY   STATUS    RESTARTS   AGE
  cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
  cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
  cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

## Issuer
Issuer 用來頒發憑證，分為兩種資源類型：
- issuer：只作用於特定 namespace
- ClusterIssuer：作用於整個 k8s 集群

cert-manager 有支援幾種的 issuer type：
- CA: 使用 x509 keypair 產生 certificate，存在 kubernetes secret
- Self Signed: 自簽 certificate
- ACME: 從 ACME (ex. Let's Encrypt) server 取得 ceritificate
- Vault: 從 Vault PKI backend 頒發 certificate
- Venafi: Venafi Cloud

[Above Refenrence](https://ithelp.ithome.com.tw/articles/10227274)

## Create Issuer

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  # 創建的簽發機構的名稱，後面創建證書的時候會引用
  name: letsencrypt-prod
spec:
  acme:
  # 證書快過期的時候會有郵件提醒，不過 cert-manager 會利用 acme 協議自動給我們重新頒發證書來續期
    email: ulahsieh@nexaiot.com
    privateKeySecretRef:
      # Name of a secret used to store the ACME account private key 指示此簽發機構的私鑰將要存儲到哪個 Secret 中
      name: letsencrypt-prod
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    solvers:
    # 指示簽發機構使用 HTTP-01 的方式進行 acme 協議(還可以用 DNS 方式，acme 協議的目的是證明這台機器和域名都是屬於你的，然後才准許給你頒發證書)
    - http01:
        ingress:
          class: istio
```

備註：

- letsencrypt 可以看到網路上有兩個不同名字的設定，一個是 letsencrypt-staging 用於測試，一個是 letsencrypt-prod 用於 production.

```bash
$ kubectl apply -f clusterIssuer.yaml
$ kubectl describe clusterissuers.cert-manager.io
```

做完的時候發生了 server misbehaving 錯誤

```bash
Events:
  Type     Reason         Age                  From          Message
  ----     ------         ----                 ----          -------
  Warning  ErrInitIssuer  8m2s (x3 over 8m7s)  cert-manager  Error initializing issuer: Get "https://acme-v02.api.letsencrypt.org/directory": dial tcp: lookup acme-v02. api.letsencrypt.org on 10.96.0.10:53: server misbehaving
```

查了一下發現是 dns 解析的問題，便去自建的 dns server 改 /etc/named.conf 檔，發現在 option 中少加了 forwarders 欄位。

forwarders 是指當本 DNS 解析不了的域名，要轉給誰來解析的意思，通常轉給再上一層，也就是外網本身的 DNS，簡單來說可直接使用 8.8.8.8，並添加 `allow-query any;`，讓集群內的網段都能來使用。

改好之後回到 k8s 集群再次查看 clusterissuer 是否可以建成

```bash
[root@k8sm1 cert-manager]# kubectl describe clusterissuers.cert-manager.io
Name:         letsencrypt-prod
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  cert-manager.io/v1
Kind:         ClusterIssuer
Metadata:
  Creation Timestamp:  2021-06-21T12:12:47Z
  Generation:          1
  Resource Version:    104200852
  Self Link:           /apis/cert-manager.io/v1/clusterissuers/letsencrypt-prod
  UID:                 f0e9ecc6-9a50-491c-af78-b88670885e18
Spec:
  Acme:
    Email:            ulahsieh@nexaiot.com
    Preferred Chain:
    Private Key Secret Ref:
      Name:  letsencrypt-prod
    Server:  https://acme-v02.api.letsencrypt.org/directory
    Solvers:
      http01:
        Ingress:
          Class:  istio
Status:
  Acme:
    Last Registered Email:  ulahsieh@nexaiot.com
    Uri:                    https://acme-v02.api.letsencrypt.org/acme/acct/127768449
  Conditions:
    Last Transition Time:  2021-06-21T12:13:35Z
    Message:               The ACME account was registered with the ACME server
    Observed Generation:   1
    Reason:                ACMEAccountRegistered
    Status:                True
    Type:                  Ready
Events:                    <none>
```

## Create Certificate
透過 Issuer 申請 Certificate 憑證
```yaml
# cat istio-cert.yaml 
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: istio-cert
  namespace: istio-system
spec:
  commonName: convergence.nexmasa.com
  dnsNames:
  - convergence.nexmasa.com
  secretName: istio-cert
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
```

說明：

- spec.secretName 指示證書最終存到哪個 Secret 中
- spec.issuerRef.kind 值為 ClusterIssuer 說明簽發機構不在本namespace 下，而是在全局
- spec.issuerRef.name 我們創建的簽發機構的 Issuer 名稱
- spec.dnsNames 指示該證書的可以用於哪些域名

## 踩坑囉

部屬 certificate 後一直卡在跟 letsencrypt issueing 這塊，錯誤訊息是:

```bash
$ kubectl describe certificate -n istio-system istio-cert
$ kubectl get event -n istio-system
30s         Normal    Issuing           certificate/istio-cert                            Issuing certificate as Secret does not exist
29s         Normal    Generated         certificate/istio-cert                            Stored new private key in temporary Secret resource "istio-cert-hrrgb"
29s         Normal    Requested         certificate/istio-cert                            Created new CertificateRequest resource "istio-cert-89tj8"
3s          Warning   Failed            certificate/istio-cert                            The certificate request has failed to complete and will be retried: Failed to wait for order resource "istio-     cert-89tj8-2838533447" to become ready: order is in "invalid" state:
```

試了很久，發現官網有寫 debug 過程，才發現要去看 challenge 的 log

[https://cert-manager.io/docs/faq/acme/](https://cert-manager.io/docs/faq/acme/)

```yaml
$ kubectl describe challenges.acme.cert-manager.io -n istio-system istio-cert-22gl9-2838533447-3373762545
...
Events:
  Type     Reason     Age   From          Message
  ----     ------     ----  ----          -------
  Normal   Started    32m   cert-manager  Challenge scheduled for processing
  Normal   Presented  32m   cert-manager  Presented challenge using HTTP-01 challenge mechanism
  Warning  Failed     32m   cert-manager  Accepting challenge authorization failed: acme: authorization error for convergence.nexmasa.com: 400 urn:ietf:params:acme:error:                                  dns: DNS problem: NXDOMAIN looking up A for convergence.nexmasa.com - check that a DNS record exists for this domain
```

然後找了一下解答發現，letsencrypt 只適合用在網際網路存取到的 DNS 啊 ... = =

[https://github.com/jetstack/cert-manager/issues/3543](https://github.com/jetstack/cert-manager/issues/3543)

> This error comes from Let's Encrypt which cannot reach your domain (it tries to as .int is a public known TLD).
In order for Let's Encrypt to work they need to have public access to verify the ownership of your domain.
For internal only domains you might want to look into using an internal CA.

[https://serverfault.com/questions/1048678/check-that-a-dns-record-exists-for-this-domain](https://serverfault.com/questions/1048678/check-that-a-dns-record-exists-for-this-domain)

> domain names that are in the global DNS tree

不過這邊還是可以記錄幾篇可以參考使用 letsencrypt 的文章

- [https://www.qikqiak.com/k8strain/istio/cert-manager/](https://www.qikqiak.com/k8strain/istio/cert-manager/)
- [https://medium.com/intelligentmachines/istio-https-traffic-secure-your-service-mesh-using-ssl-certificate-ac20ec2b6cd6](https://medium.com/intelligentmachines/istio-https-traffic-secure-your-service-mesh-using-ssl-certificate-ac20ec2b6cd6)


## 改建自簽憑證
- 建立 Issuer
```yaml
kubectl apply -f <(echo "
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned
  namespace: istio-system
spec:
  selfSigned: {}
")
```
- 建立 Certificate
```yaml=
kubectl apply -f <(echo '
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: selfsigned-crt
  namespace: istio-system
spec:
  secretName: tls-secret
  duration: 175200h
  renewBefore: 12h
  issuerRef:
    kind: Issuer
    name: selfsigned
  commonName: "convergence.nexmasa.com"
  isCA: true
  dnsNames:
  - "convergence.nexmasa.com"
')
```
- 查看憑證及金鑰的有效性
```bash
kubectl get secrets/tls-secret -n istio-system -o "jsonpath={.data['tls\.crt']}" | base64 -d | openssl x509 -text -noout
```

</br>

![](https://imgur.com/yx8o1Xn.png)

</br>

```bash
kubectl get secrets/tls-secret -n istio-system -o "jsonpath={.data['tls\.key']}" | base64 -D | openssl rsa -check
```

</br>

![](https://imgur.com/EQdAiCO.png)

- 為服務加上憑證
修改 istio gateway，加上 https 的 protocol 並指定上面建立的 Secret Name。
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: converg-api-gw
  namespace: converg-api
spec:
  selector:
    istio: ingressgateway
  servers:
  - hosts:
    - '*'
    port:
      name: Port-80-M55Sa
      number: 80
      protocol: HTTP
  - hosts:
    - '*'
    port:
      name: https
      number: 443
      protocol: HTTPS
    tls:
      credentialName: tls-secret
      mode: SIMPLE
```