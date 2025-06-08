---
title: "[Zookeeper] ACL"
categories: ["DataEngineering", "ApacheKafka"]
tags: ["Zookeeper"]
date: 2020-08-12 20:45:00
slug: "zookeeper-acl"
---

## ACL(Access Control List)

Zookeeper 對 znode 操作採用 ACL 進行了存取權限控制，類似於 UNIX/Linux 的文件權限機制。使用 scheme&#058;id&#058;perm 來標識，主要涵蓋 3 個方面：

<!--more-->

- 權限模式（Scheme）：授權的策略
- 授權對象（ID）：授權的對象
- 權限（Permission）：授予的權限  
  其特性如下：
- ZooKeeper 的權限控制是基於每個 znode 節點的，需要對每個節點設置權限
- 每個 znode 支持設置多種權限控制方案和多個權限
- 子節點**不會**繼承父節點的權限，客戶端無權訪問某節點，但可能可以訪問它的子節點

---

### Scheme:id

ZK 支持 pluggable authentication schemes，可以通過擴充套件 scheme，來擴充 ACL 的機制。

#### Built-in ACL Schemes

ZK 有以下的内置 schemes：

- world：默認方式，僅對應一個 id `anyone`，為所有用戶端開發權限。
- auth：代表**已經**認證通過的用戶 (cli 中可以通過 `addauth digest user:pwd` 來添加當前上下文中的授權用戶；或是通過 kerberos 來進行 authencation)
- digest：即`用戶名:密碼`這種方式認證。用 `username:BASE64(SHA1(password))`  字符串作為 ACL ID。意思是認證是通過明文發送 `username:password`  來進行的，當用在 ACL 時，表達式為 `username:BASE64(SHA1(password))`。
- ip：使用客戶端的主機 IP 作為 ACL ID 。可以使用一個網段，如 10.15.0.0/16。
- super：在這種 scheme 情況下，對應的 id 擁有超級許可權，可以做任何事情 (cdrwa)。
- sasl：設置為用戶的 uid，通過 sasl Authentication 用戶的 id，在 3.4.4 版本後 sasl 是通過 Kerberos 實現（即只有通過 Kerberos 認證的用戶才可以訪問權限的 znode），使用 `sasl:uid:cdwra` 字符串作為節點 ACL 的 id（如：sasl:user:cdwra）。

### Permission

5 種權限簡寫為 crwda

- CREATE c 可以創建當前節點的**子節點**
- DELETE d 可以刪除當前節點的**子節點**
- READ r 可以讀取**節點**數據及顯示**子節點**列表
- WRITE w 可以向當前**節點**寫數據
- ADMIN a 可以設置**節點**訪問控制列表權限

---

### ACL 相關命令

| 命令    | 使用方式                  | 描述          |
| ------- | ------------------------- | ------------- |
| addauth | `addauth <scheme> <auth>` | 添加認證用戶  |
| setAcl  | `setAcl <path> <acl>`     | 設置 ACL 權限 |
| getAcl  | `getAcl <path>`           | 讀取 ACL 權限 |

進入 zkCli

#### auth 舉例

```
$ addauth digest user:123456
$ setAcl /test auth:user:cdrwa
```

之後使用 getAcl 查看權限時，密碼會自動轉為密文`BASE64(SHA1(password))`。

![](https://imgur.com/IBGSdHf.png)

查看節點數據之前，必須確保已驗證加入，否則會有 `Authentication is not valid : /test` 的錯誤。

#### digest 舉例

digest 加密模式相對於 auth 來說要稍微麻煩一些，需要對明文密碼進行 `BASE64(SHA1(password))` 的處理。

```
$ setAcl <path> digest:<user>:<密文密碼>:<acl>
```

##### 對密碼進行加密的方法

- 使用 openssl 命令產生

```
echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
```

![](https://imgur.com/sdvKo1X.png)

生成 `<user>:<password>` 對應的密文。請注意，如果將 user 改掉，則生成的密文也會不一樣。

- 使用 zookeeper 提供的 java class 來產生
  先進入 zookeeper 的安裝目錄，然後執行下述命令：

```
$ java -cp ./lib/*:./* org.apache.zookeeper.server.auth.DigestAuthenticationProvider ula:123456
```

![](https://imgur.com/sdvKo1X.png)

**使用 digest 可直接 setAcl 設置權限，不用添加身份認證。但如果要訪問節點，仍需先身分認證**

![](https://imgur.com/xOpBvQI.png)

---

### 設置超級管理員

假如忘記了認證用戶的密碼，或者想要存取某些被保護的 znode，怎麼辦呢？可以為 zookeeper 設置超級管理員，superuser 預設對所有節點有權限存取。

1. 取得欲設置帳號對應的密碼的密文，同上一小節**對密碼進行加密的方法**
2. 編輯啟用 zookeeper service 的 script `zkServer.sh`，將以下參數加入下方圖片中位置或是 export 為環境變數。

```
"-SERVER_JVMFLAGS=-Dzookeeper.DigestAuthenticationProvider.superDigest=[帳號]:[密文密碼]
```

![](https://imgur.com/vwyzUie.png)

3. 保存文件，重啟該節點上的 zookeeper service 便設置成功了。

4. 進入 zkCli 模式後執行

```
addauth digest [帳號]:[明文密碼]
```

認證身份，這樣就具備超級管理員角色，可以操作任意節點了。

---

{{< notice info >}}
補充
在 Linux 中執行某些程序前會對啟動它的用戶進行認證，符合一定的要求之後才允許執行，例如 login, su 等。在 linux 中進行身份或是狀態的驗證程序是由 PAM 來進行的。
PAM（Pluggable Authentication Modules）是由 Sun 提出的一種認證機制。它通過提供一些動態鏈接庫和一套統一的 API，將系統提供的服務和該服務的認證方式分開，使得系統管理員可以靈活地根據需要給不同的服務配置不同的認證方式而無需更改服務程序，同時也便於向系統中添加新的認證手段。PAM 模塊是一種嵌入式模塊，修改後即時生效。
{{< /notice >}}

## Reference

- https://baike.baidu.com/item/PAM/3747946
- https://www.cnblogs.com/qlqwjy/p/10517231.html
- https://cloud.tencent.com/developer/article/1414462
