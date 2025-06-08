---
title: "[Zookeeper] Encryption - node-node & client-node"
categories: ["DataEngineering", "ApacheKafka"]
tags: ["Zookeeper"]
date: 2020-08-30 10:10:00
slug: "zookeeper-encryption-3"
---

## Node-Node Encryption

節點到節點加密使用 SSL 保護 ZooKeeper server 間的內部連接，加密完全在 ZooKeeper 節點之間完成。默認情況下，Quorom TLS 是禁用的，必須通過編輯所有 server 中的 `zoo.cfg` 文件來啟用：

<!--more-->

```
# zoo.cfg
sslQuorum=true
serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
ssl.quorum.keyStore.location=<path to keystore>
ssl.quorum.keyStore.password=<password>
ssl.quorum.trustStore.location=<path to truststore>
ssl.quorum.trustStore.password=<password>
```

接著重新啟用 zookeeper 集群，查看 Zookeeper server log 出現以下訊息，便可以使用 ssl 在 server 間通訊。

```
info [main:QuorumPeer@1789] - Using TLS encrypted quorum communication info [main:QuorumPeer@1797] - Port unification disabled ... info [QuorumPeerListener:QuorumCnxManager$Listener@877] - Creating TLS-only quorum server socket
```

---

## Client-Node Encryption

1. 針對 server 端，編輯 zkServer.sh 檔案，加入以下環境變數

```
export SERVER_JVMFLAGS="
-Dzookeeper.serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
-Dzookeeper.ssl.keyStore.location=<path to keystore>
-Dzookeeper.ssl.keyStore.password=<password>
-Dzookeeper.ssl.trustStore.location=<path to truststore>
-Dzookeeper.ssl.trustStore.password=<password>"
```

2. 編輯 zoo.cfg 檔案，加入 secureClientPort

```
...
secureClientPort=2281
```

3. 針對 client 端，以 zkCli 為例，編輯 zkCli.sh 加入以下環境變數

```
export CLIENT_JVMFLAGS="
-Dzookeeper.clientCnxnSocket=org.apache.zookeeper.ClientCnxnSocketNetty
-Dzookeeper.client.secure=true
-Dzookeeper.ssl.keyStore.location=<path to keystore>
-Dzookeeper.ssl.keyStore.password=<password>
-Dzookeeper.ssl.trustStore.location=<path to truststore>
-Dzookeeper.ssl.trustStore.password=<password>"
```

4. 重啟 Zookeeper，並透過 secure client port 使用 zkCli 連進 Zookeeper server。

```
/opt/zookeeper/bin/zkServer.sh restart /opt/zookeeper/conf/zoo.cfg
/opt/zookeeper/binzkServer -server 10.1.5.31:2281
```

## 補充：如何使用 ssl 在 zkCLi 中使用 dynamic reconfig 的功能

在這篇[文章](https://ulahsieh.netlify.app/p/zookeeper-dynamic-configuration/)中有提到 reconfig 存取的 znode 需要有 ACL 讀寫的權限或是 super user 的身分。
透過 ssl 使用 zkCLi 加密連線登入時，需要使用 `X509AuthenticationProvider.superUser` 參數，值設為 X500 principal name，將 x509 金鑰使用者加入 super user。

```
# zoo.cfg
-Dzookeeper.X509AuthenticationProvider.superUser=CN=pc31
```

## Reference

- https://zookeeper.apache.org/doc/r3.6.1/zookeeperAdmin.html#Quorum+TLS
- https://cwiki.apache.org/confluence/display/ZOOKEEPER/ZooKeeper+SSL+User+Guide
