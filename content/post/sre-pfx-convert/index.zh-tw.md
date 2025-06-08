---
title: "[SSL] PFX 轉 PEM, CRT, KEY & s_client, s_server 測試"
categories:
  - SRE
tags: ["SSL"]
slug: "ssl-pfx-to-pem"
date: 2020-07-20 16:48:00
---

憑證發行機構(CA)發出的 PFX 憑證文件，通常將根憑證/中繼憑證、網站憑證(certificate)和網站金鑰(網站私鑰)集合在一起。

<!--more-->

### pfx 轉 pem (CA)

```
$ openssl pkcs12 -in keyname.pfx -out keyname.pem -nodes
```

### pfx 轉 crt (cert)

```
$ openssl pkcs12 -in keyname.pfx -nokeys -clcerts -nodes -out keyname.crt
```

### pfx 轉 key

```
$ openssl pkcs12 -in keyname.pfx -nocerts -nodes -out keyname.key
```

以上三個步驟皆須再輸入當時匯出 pfx 設定的密碼。

- -in filename：指定私鑰和證書讀取的文件，默認為標準輸入。
- -out filename：指定輸出的文件，默認為標準輸出。
- -clcerts：僅僅輸出客戶端證書，不輸出 CA 證書。
- -cacerts：僅僅輸出 CA 證書，不輸出客戶端證書。
- -nocerts：不輸出任何證書。
- -nokeys：不輸出任何私鑰信息值。
- -nodes：對私鑰不加密。

---

## 查看憑證 / 金鑰內容

### 查看憑證 pem

```
$ openssl x509 -in keyname.pem -text -noout
```

### 查看證書 cert

```
$ openssl x509 -noout -text -in keyname.crt
```

### 查看金鑰 key

```
$ openssl rsa -noout -text -in keyname.key
```

---

## 測試工具

Openssl 提供了簡單的 client 和 server 工具，可以用来模擬 SSL 連接，以測試診斷。

### s_client

以 ssl 協議連接到遠程伺服器，

```
$ openssl s_client -connect [host]:443
```

- -connect host:port：指定遠程服務器的地址和端口，默認值為 localhost:443
- -CAfile filename：指定用於驗證服務器證書的根證書
- -cert filename：若服務器端需要驗證客戶端的身份，通過 -cert 指定客戶端的證書文件
- -key filename：指定私鑰文件
- -state：打印出 SSL 會話的狀態

### s_server

模擬 HTTPS 服務，可以返回 Openssl 相關訊息

```
$ openssl s_server -accept 443 -cert myserver.crt -key myserver.key -www
```

- -accept：用来指定監聽的 port
- -cert：用来指定提供服務的證書
- -key：用来指定提供服務的 key

---

## Source

- [openssl 查看证书](https://www.jianshu.com/p/f5f93c89155e)
- [pfx 分解成 pem](https://cloud.tencent.com/developer/article/1556287)
