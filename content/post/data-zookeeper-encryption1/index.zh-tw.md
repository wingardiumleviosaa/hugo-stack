---
title: "[Zookeeper] Encryption - 產生 self-signed 金鑰"
categories: ["DataEngineering", "ApacheKafka"]
tags: ["Zookeeper"]
date: 2020-08-30 09:38:00
slug: "zookeeper-encryption-1"
---

Netty 是一個基於 NIO 的 client/server 通信框架，它通過直接使用 NIO 簡化了 Java 應用程序的 network level 通信的複雜度。 此外，Netty 框架內置了對加密（SSL）和身份驗證（CA）的支持。 這些是可選功能，可以單獨打開或關閉。

<!--more-->

Zookeeper 3.5 版開始，通過將環境變數的設定允許 ZooKeeper server 使用 Netty 代替 NIO（默認項）。

接下來將用幾篇文章記錄如何加密 Zookeeper 通訊。

## 產生金鑰

每個 server 必須具有包含金鑰證書（private key + public certificate）的金鑰庫 (keystore)。 金鑰證書可以是 self-signed 的，也可以由證書頒發機構（CA）簽名。

### 產生 self-signed 金鑰

1. 進入 Zookeeper server 所在機器
2. 使用 Java keytool 工具產生 keypair

```
keytool -genkeypair -alias $(hostname -f) \
  -keyalg RSA -keysize 2048 \
  -dname "CN=$(hostname -f)"
  -validity <days> \
  -keypass <password> \
  -keystore keystore.jks \
  -storepass <same password> \
  -storetype JKS
```

- hostname -f 顯示主機的 FQDN (fully qualified domain name) 完全限定域名。
- 別名（-alias）和專有名稱（-dname）必須與與之關聯的 server hostname 匹配，否則主機名驗證會失敗。

3. 從 keystore 中導出證書

```
keytool -exportcert -alias $(hostname -f) \
  -keystore keystore.jks \
  -file $(hostname -f).cer -rfc
```

4. 在 Zookeeper quorum 重複以上動作。

## 將證書放入 truststore 中

為了使 server 間彼此信任，它們必須在自己的信任庫(truststore)中具有其他 server 的證書。 Truststore 可用於 client-to-node 加密以及 node-to-node 的加密。

1. 使用 Java keytool 工具將所有 Zookeeper 實例的 certificate 導入至 truststore 中

```
keytool -importcert -alias [host1..3]
  -file [host1..3].cer \
  -keystore truststore.jks \
  -storepass <password>
```

- alias：別名可以是任何值。可以與 keystore 使用相同別名，也可是諸如 self 之類的描述性標籤。
- file：指出愈導入的公開證書

2. 出現提示時，輸入 `y` 將證書添加到信任庫中：

```
Trust this certificate? [no]:  y
Certificate was added to keystore
```

3. 重複以上動作，為在 quorum 的 Zookeeper 建立 truststore。

## Reference

- https://zookeeper.apache.org/doc/r3.6.1/zookeeperAdmin.html#Quorum+TLS
