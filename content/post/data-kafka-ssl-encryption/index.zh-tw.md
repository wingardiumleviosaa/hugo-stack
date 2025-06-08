---
title: "[Kafka] server - client & client - server SSL 加密"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Kafka"]
date: 2020-09-04 12:00:00
slug: "kafka-ssl-encryption"
---

Kafka 目前支持 SSL、SASL/Kerberos、SASL/PLAIN 三種認證機制，這篇文章將紀錄如何在 Kafka broker 與 clinet 端建立 TLS/SSL 連線。

<!--more-->

另外關於 SASL 連線機制，可以參考 [官網文件](https://kafka.apache.org/25/documentation.html#security_sasl) 或是 [中文版文件](https://www.orchome.com/553)。

步驟如下：

## 生成密鑰/ 證書/ CA

### 在各個實例中生成 keystore

需要在集群中的每台機器上單獨生成，keystore 包含了兩個東西：

- 含有公鑰(Public Key)/私鑰(Private Key) 的 Key pair
- 只包含公鑰的未簽名證書(Unsigned Certificate)

```
$ keytool -keystore broker1.keystore.jks \
  -alias broker1 -validity 3650 -genkey -keyalg RSA
```

產生過程中需輸入 keystore 的密碼以及證書的 Distinguished Name (DName)。其中最重要的是 `Common Name(CN)`，需為機器的 hostname；如果主機綁定了域名，則輸入完整包含域名的 FQDN。

![](https://imgur.com/Y4sFl2r.png)

在其他 broker 以及 clinet 端重複以上動作。

{{< notice info >}}
證書相當於一個 ID，來表明自己是誰（區分不同的服務器）。此時，這個證書還沒有被任何 CA（Certificate Authority，證書頒發機構）所認證（進行簽名）。證書可以通過下面兩種方式簽名：

- 先自己生成一個證書，再讓 CA 簽名。(此篇將採用的方法)
- 直接從 CA 申請一個。
  {{< /notice >}}

---

### 在任一台機器中產生 CA

通常情況下，證書是要向 CA 申請的，但是如果參與者僅是自家客戶端，而非面向網際網路上的客戶端（例如瀏覽器），那麼可以自己生成一個 CA，然後讓這個 CA 去簽署其他證書。

```
$ openssl req -new -x509 -keyout ca-key -out ca-cert -days 3650
```

產生過程中需輸入 PEM 密碼，以及證書相關資訊。而後會產生 CA 密鑰 `ca-key` 以及公鑰 `ca-cert`。

![](https://imgur.com/2ksa435.png)

### 生成 truststore

truststore 包含了所有可以信賴的 CA。以下面腳本創建 truststore，並將上一步驟生成的 ca-cert 匯入。

```
keytool -keystore ca.truststore.jks -alias CARoot -import -file ca-cert
```

並將這一個 truststore 傳送到叢集的所有機器中以確保：

{{< notice warning >}}
當 client 與 broker 溝通時，client 只要檢查 broker 端的證書，確認該證書是由自己擁有的 truststore 中的 CA 頒發的，就認為此 broker 是可信的。  
同理，當 broker 與 client 交互時，broker 只要檢查 client 端證書，確認該證書是由自己所有的 truststore 中的 CA 頒發的，就認為客戶端是可信的。
{{< /notice >}}

{{< notice info >}}
如果用不同的 CA 簽署證書，那麼就要將對象端的證書，或者該 CA，加入到本地端的 truststore 中。  
這裡有一個信任鏈的關係：可以信任某一個實例的證書；也可以信任證書的頒發機構，即 CA，當信任 CA 時，就會信任所有由該 CA 頒發的證書。  
**因此，在內部使用的叢集中，所有的伺服器只要信任同一個 CA，使用同一個 truststore 就可以了（那麼所有持有由該 CA 簽名的證書的服務器就都是可信的了）。**
{{< /notice >}}

<font color=carol>將 CA 以及 ca.truststore.jks 複製到其他 broker 以及 clinet 端，使其共用同一把 CA。</font>

---

### 將各伺服器的證書導出並使用 CA 簽名

從第一步驟的 keystore 中導出未簽名的證書

```
keytool -keystore broker1.keystore.jks -alias broker1 -certreq -file cert-broker1
```

使用 CA 對導出的 `cert-broker1` 進行簽名

```
openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-broker1 -out cert-signed-broker1 -days 3650 -CAcreateserial -passin pass:<PEM password>
```

將 CA 證書以及簽名好的 `cert-signed-broker1` 證書導回至 `keystore`
**注意先後順序**

```
$ keytool -keystore broker1.keystore.jks -alias CARoot -import -file ca-cert
$ keytool -keystore broker1.keystore.jks -alias broker1 -import -file cert-signed-broker1
```

在其他 broker 以及 clinet 端重複以上動作。

最後架構中的金鑰結構會長得像下面這樣

![](https://imgur.com/gMdBG1h.png)

---

## 配置 broker 端

### 修改 Kafka 的配置檔 `server.properties`

```
listeners=PLAINTEXT://0.0.0.0:9092,SSL://0.0.0.0:9093
advertised.listeners=PLAINTEXT://broker1:9092,SSL://broker1:9093

# SSL 配置
ssl.keystore.location=/opt/ssl/broker1.keystore.jks
ssl.keystore.password=000000
ssl.key.password=000000
ssl.truststore.location=/opt/ssl/ca.truststore.jks
ssl.truststore.password=000000
ssl.keystore.type=JKS
ssl.truststore.type=JKS
ssl.client.auth=required
ssl.secure.random.implementation=SHA1PRNG
ssl.endpoint.identification.algorithm=
```

- ssl.client.auth： `required` 這個選項則要求客戶端也必須持有經過 CA 簽名的證書； `requested` 要求進行客戶端身份認證但沒有證書的客戶端也可以連接；`none` 默認值，指服務端不須驗證客戶端。
- ssl.secure.random.implementation：用來指定資料加密方法。
  JRE/JDK 有默認的偽隨機數生成器（PRNG）用於加密操作，因此不需要使用 ssl.secure.random.implementation 來配置具體的實現方法。但是，某些實現方法是有性能問題的（特別是 Linux 系統上的默認選項：NativePRNG, 利用的是一個全局鎖），為以防 SSL 連接性能成為問題，請考慮明確設置使用的實現方法。 SHA1PRNG 方式的實現是非阻塞的，在高負載的情況下也能有很好的性能表現（單 broker 在有副本的情況下達 50MB 每秒的生產消息）。
- ssl.endpoint.identification.algorithm：kafka 2.0 開始的默認值是 https，即需要驗證主機名。如果不需要驗證主機名，那麼可以這麼設置 等於空即可。

**另外還有一個選項 `security.inter.broker.protocol` 這邊沒有配置，如果要指定 broker 間也要使用 SSL，則將值設為 `SSL`（默認為　 PLAINTEXT）。**
因為集群內的所有 broker 都位於內網，所以這個例子沒有特別開啟，這也是上面為什麼 listeners 要同時配置 PLAINTEXT 和 SSL 的原因。

### 啟動 Kafka 並測試連線

運行服務端

```
bin/kafka-server-start.sh -daemon config/server.properties
```

再通過下面的命令來驗證 SSL 連接是否運行正常

```
openssl s_client -debug -connect localhost:9093 -tls1
```

成功的話可以看到以下訊息

![](https://imgur.com/FNmDaCa.png)

---

## 配置 client 端

新增 `client-ssl.properties` 檔案，並設置以下參數：

```
security.protocol=SSL
ssl.keystore.location=/opt/ssl/client1.keystore.jks
ssl.keystore.password=000000
ssl.truststore.location=/opt/ssl/ca.truststore.jks
ssl.truststore.password=000000
ssl.key.password=000000
ssl.endpoint.identification.algorithm=
```

---

## 測試連線

```
bin/kafka-console-producer --bootstrap-server broker1:9093 --topic test --producer.config client-ssl.properties
bin/kafka-console-consumer --bootstrap-server broker1:9093 --topic test --consumer.config client-ssl.properties --from-beginning
```

![](https://imgur.com/rZR6yHh.png)

---

## Reference

- https://kafka.apache.org/25/documentation.html#security
- https://www.denglin.me/kafka-security-1/
- http://www.tracefact.net/tech/108.html
