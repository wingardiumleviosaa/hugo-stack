---
title: "[Zookeeper] Dynamic Configuration"
categories: ["DataEngineering", "ApacheKafka"]
tags: ["Zookeeper"]
date: 2020-08-12 22:10:00
slug: "zookeeper-dynamic-configuration"
---

Zookeeper 在 3.5.0 版發行後，支持動態新增節點，不需將整個集群重啟。這篇文章主要紀錄 dymamic configuration 如何實作。

<!--more-->

## 環境準備

準備三台 VM，[安裝 Zookeeper](https://ulahsieh.netlify.app/p/zookeeper-build-cluster/)

- 10.1.5.31
- 10.1.5.32
- 10.1.5.33

## 配置 zoo.cfg

配置三台 Zookeeper 的配置檔，新增參數說明如下：

```
dataDir=/var/lib/zookeeper
syncLimit=5
tickTime=2000
initLimit=10
maxClientCnxns=100
standaloneEnabled=false
reconfigEnabled=true
4lw.commands.whitelist=*
dynamicConfigFile=/opt/zookeeper/conf/zoo.cfg.dynamic
```

- standaloneEnabled  
  在 3.5.0 版之前，可以在獨立模式或分佈式模式下運行 ZooKeeper。並且無法在運行時在它們之間進行切換。 默認情況下 standaloneEnabled 設置為 true。使用默認值的話，則當以單個服務器啟動時，不允許集合增長；而如果是以叢集方式啟動，則不允許減少小於等於兩個參與者。由於運行分佈式模式可以提供更大的靈活性，因此建議將標誌設置為 false。

- reconfigEnabled  
  從 3.5.0 版開始到 3.5.3 之前，動態重新配置功能皆無法禁用，涉及安全問題，惡意行為者可以對 ZooKeeper 集合的配置進行任意更改，或是將受感染的服務器添加到集合中。
  3.5.3 新增了這個參數，並加入一些安全機制（請見下方說明），讓使用者自行決定是否啟用。reconfigEnabled 設為 false 可以完全禁用重新配置功能，默認情況下，無論是否使用身份驗證通過 reconfig API 重新配置群集的任何嘗試都將失敗，除非 reconfigEnabled 設置為 true 。

- dynamicConfigFile  
  從 3.5.0 版開始，zookeeper 區分`動態配置參數`和`靜態配置參數`，靜態配置參數在 servie 啟動時從配置文件 zoo.cfg 中讀取，並且在執行期間不會更改。動態配置參數則可以寫入 dynamicConfigFile 中。<font style="background:MistyRose">目前動態配置的參數有 server, group and weight。</font>動態參數將由 ZooKeeper 推送，並覆蓋所有服務器上的動態配置文件。因此，不同服務器上的動態配置文件通常是相同的（它們只能在重新配置進行時暫時不同，或者如果新配置尚未傳播到某些服務器）。創建後，不應手動更改動態配置文件。只可以通過 API 或者 reconfig 命令進行更改。

## 配置 zoo.cfg.dynamic

```
server.1 = 10.1.5.31:2888:3888:participant;2181
server.2 = 10.1.5.32:2888:3888:participant;2181
server.3 = 10.1.5.33:2888:3888:participant;2181
```

格式為
`server.<id> = <address1>:<port1>:<port2>[:role];[<client port address>:]<client port>`

- 第一個 port 是用來和集群中的 Leader 交換訊息的；
- 第二個 port 是在 leader 掛掉時用來進行選舉的。
- 角色是可選的，可以是 participant (default) 或者 observer。
- client ip & port 放在最後面且用分號分開。client IP 是可選的，默認是`0.0.0.0`。

下面是合法的範例：

```
server.5 = 125.23.63.23:1233:1235;2181
server.5 = 125.23.63.23:1233:1235:participant;2181
server.5 = 125.23.63.23:1233:1235:observer;2181
server.5 = 125.23.63.23:1233:1235;125.23.63.24:2181
server.5 = 125.23.63.23:1233:1235:participant;125.23.63.23:2181
```

myid 的數字<span class="dotunderletter">**不一定**</span>是從 1 開始依序排序，可以自己指定，但不同的服務器的 id 要唯一。

## 初始化 myid

在每台 Zookeeper 的 dataDir 目錄分別執行：

```
# 10.1.5.31
echo "1">/var/lib/zookeeper/myid

# 10.1.5.32
echo "2">/var/lib/zookeeper/myid

# 10.1.5.33
echo "3">/var/lib/zookeeper/myid
```

## 啟動服務

在三台機上上啟動 Zookeeper

```
/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg
```

進入其中一台 Zookeeper CLI，比如 10.1.5.31，

```
/opt/zookeeper/bin/zkCli.sh -server 10.1.5.31:2181
```

![](https://imgur.com/ThMerfx.png)

這裡可以發現集群訊息中有個 `version=xxxxxxxxx` 的配置，這是動態配置文件的版本號，在 zookeeper 的 conf 文件夾下，你會發現多了一個配置文件 `zoo.cfg.dynamic.xxxxxxxxx`，說明集群當前使用的是這個配置文件，會依每次動態配置後而改動。

## 安全性

在 3.5.3 版之前，任何可以連接到 ZooKeeper 集群的 client 都可以通過 reconfig 來更改 ZooKeeper 集群的狀態，進而使惡意客戶端有機可趁。因此 3.5.3 開始引入了對 reconfig 或 API 訪問的控制。
動態配置存儲在特殊的 znode，ZooDefs.CONFIG_NODE = /zookeeper/config 中。默認情況下，此節點對所有用戶都是 `read-only` 的，但超級使用者和 CONFIG_NODE 配置為`寫`訪問權限的 user 除外。
關於如何修改 ACL 權限，請參考上一篇文章 － [Zookeeper ACL](https://ulahsieh.netlify.app/p/zookeeper-acl/)

{{< notice warning >}}
另外也可以使用 `skipACL` 來禁用 ACL 檢查，ZooKeeper 支持在　 zoo.cfg 中的 `skipACL` 參數設置為 `yes` 時完全跳過 ACL。在這種情況下，任何未經身份驗證的用戶都可以使用 reconfig API。不安全，建議不要使用。
{{< /notice >}}

## 動態增加 & 刪除節點

### 新增節點

準備另一台機器 10.1.5.90，安裝 zookeeper 後，靜態配置檔 zoo.cfg 如同上方配置。動態配置檔 zoo.cfg.dynamic 需新增自己的資訊：

```bash
server.1=10.1.5.31:2888:3888:participant;2181
server.2=10.1.5.32:2888:3888:participant;2181
server.3=10.1.5.33:2888:3888:participant;2181
server.4=10.1.5.90:2888:3888:participant;2181
```

進入前三台中的任一台 zkCli 後執行

```
reconfig -add server.4=10.1.5.90:2888:3888:participant;2181
```

![](https://imgur.com/a3WeNXb.png)

### 刪除節點

```
reconfig -remove 4
```

![](https://imgur.com/2pOb4s9.png)

### 確認新增節點是否搭建成功

確認節點狀態

![](https://imgur.com/0q26paF.png)

在 10.1.5.31 新增一個 znode

![](https://imgur.com/Be4DUG6.png)

到 10.1.5.90 確認是否有同步成功

![](https://imgur.com/rt7IiHJ.png)

## Reference

- [zookeeperReconfig1](https://github.com/apache/zookeeper/blob/master/zookeeper-docs/src/main/resources/markdown/zookeeperReconfig.md)
- [zookeeperReconfig2](https://docs4dev.com/docs/zh/zookeeper/r3.5.6/reference/zookeeperReconfig.html#sc_reconfig_access_control)
- [zookeeper3.6.1 安装配置](https://blog.csdn.net/zhoulenihao/article/details/107076188)
