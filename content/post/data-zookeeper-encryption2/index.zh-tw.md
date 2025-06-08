---
title: "[Zookeeper] Encryption - 產生 CA-signed 金鑰"
categories: ["DataEngineering", "ApacheKafka"]
tags: ["Zookeeper"]
date: 2020-08-30 09:47:00
slug: "zookeeper-encryption-2"
---

## 產生 CA 加簽金鑰

1. 產生 CA 金鑰憑證
在隨意一台機器上產生 CA 憑證，用於為其他金鑰加簽。
<!--more-->

```
openssl req -new -x509 \
  -keyout test.ca.key \
  -out test.ca.crt \
  -days <days> \
  -passout pass:<password> \
  -subj "/C=TW/ST=NewTaipei/L=NewTaipei/O=ORG/OU=test/CN=localhost"
```

會產生加密的 RSA 密鑰 `test.ca.key` 以及加密的公開 CA 憑證 `test.ca.crt`

2. 產生 keypair
   在每一台要加入 SSL 的機器上使用 Java keytool 工具產生屬於自己的 keypair。

```
keytool -genkeypair \
  -alias $(hostname -f) \
  -keyalg RSA -keysize 2048 \
  -dname "CN=$(hostname -f)" \
  -validity <days> \
  -keypass <password> \
  -keystore keystore.jks \
  -storepass <same password> \
  -storetype JKS
```

3. 將第一步驟產生的 CA 憑證加到個別機器上的 truststore

```
keytool -importcert \
  -keystore truststore.jks \
  -file test.ca.crt \
  -alias CARoot \
  -keypass <password> \
  -storepass <password> \
  -noprompt
```

4. 從第二步驟產生的 keypair keystore.jks 中匯出 public certificate

```bash
keytool -certreq \
  -keystore keystore.jks \
  -file $(hostname).crt \
  -alias $(hostname) \
  --keypass <password> \
  --storepass <password>
```

5. 使用 CA 簽署第四步驟產生的憑證

```bash
openssl x509 -req \
  -CA test.ca.crt \
  -CAkey test.ca.key \
  -in $(hostname).crt \
  -out $(hostname).signed.crt \
  -days <days> \
  -CAcreateserial \
  -passin pass:<password>
```

完成後會看到以下訊息：

```
Signature ok
subject=CN = <hostname>
Getting CA Private Key
```

6. 將第五步產生的 signed certificate 匯入到自己的 keystore 中

```bash
keytool -importcert \
  -keystore keystore.jks \
  -file $(hostname).signed.crt \
  -alias $(hostname) \
  -keypass <password> \
  -storepass <password> \
  -noprompt
```

7. 將第五步產生的 signed certificate 匯入到其他機器的 truststore 中，便可以使用 TLS 與其他機器連接。

```
keytool -importcert \
  -keystore truststore.jks \
  -file $(hostname).signed.crt \
  -alias CARoot \
  -keypass <password> \
  -storepass <password> \
  -noprompt
```

## Reference

- https://wenku.baidu.com/view/2f1a69078e9951e79b8927df.html
