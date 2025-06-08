---
title: "[Zookeeper] 建立 Zookeeper Cluster"
categories:
  - DataEngineering
  - ApacheKafka
tags: ["Zookeeper"]
date: 2020-04-30 20:55:00
slug: "zookeeper-build-cluster"
---

Zookeeper 提供分散式應用程式中的協調服務，是 Hadoop 生態系產品中不可或缺的角色。
Zookeeper 叢集適合搭建在奇數臺機器上，目的是為了提高可用性以及維持選舉制度的運行。[1]

<!--more-->

## Step 1: Install Java JDK

```sh
sudo apt-get update
sudo apt-get install openjdk-8-jdk
java -version
```

</br>

```bash {linenostart=4}
sudo vi /etc/environment
```

</br>

```
JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
JRE_HOME="/usr/lib/jvm/java-8-openjdk-amd64/jre"
PATH="${PATH}:${JAVA_HOME}/bin:${JRE_HOME}/bin"
```

使環境變數生效

```bash {linenostart=5}
source /etc/environment
env
```

## Step 2: Install zookeeper

```bash
cd /opt
wget "https://archive.apache.org/dist/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz"
sudo tar zxvf apache-zookeeper-3.6.1-bin.tar.gz
sudo mv apache-zookeeper-3.6.1-bin zookeeper
sudo adduser nexcom root
chown -R $USER:root /opt/zookeeper
```

建立存放 zk config 及其資料的 directory

```bash {linenostart=7}
mkdir /var/lib/zookeeper
chown -R $USER:root /var/lib/zookeeper
```

設定 zoo.cfg 配置檔

```bash {linenostart=9}
sudo cp /opt/zookeeper/conf/zoo_sample.cfg ./zoo.cfg
vi /opt/zookeeper/conf/zoo.cfg
```

在配置檔中修改以下參數

```yaml
dataDir=/var/lib/zookeeper
clientPort=2181

#加入其他 zookeeper 節點的 ip, 第一個 port 用於 follower 連接 leader, 第二個 port 用於節點選舉
server.1=10.0.0.4:2888:3888
server.2=10.0.0.5:2888:3888
server.3=10.0.0.6:2888:3888
```

建立 deamon，將 zookeeper 設為一服務

```bash {linenostart=11}
sudo vi /etc/systemd/system/zookeeper.service
```

在 zookeeper.service 中加入以下文字

```
[Unit]
Description=Zookeeper Daemon
Documentation=http://zookeeper.apache.org
Requires=network.target
After=network.target

[Service]
Type=forking
WorkingDirectory=/opt/zookeeper
User=root
ExecStart=/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg
ExecStop=/opt/zookeeper/bin/zkServer.sh stop /opt/zookeeper/conf/zoo.cfg
ExecReload=/opt/zookeeper/bin/zkServer.sh restart /opt/zookeeper/conf/zoo.cfg
TimeoutSec=30
Restart=on-failure

[Install]
WantedBy=default.target
```

修改 `zkServer.sh` 以避免在 telnet 時發生 `stat is not executed because it is not in the whitelist. Connection closed` 的錯誤

```bash {linenostart=12}
sudo vi /opt/zookeeper/bin/zkServer.sh
```

在 `zkServer.sh` 中加入以下這一行

```
ZOOMAIN="-Dzookeeper.4lw.commands.whitelist=* ${ZOOMAIN}"
```

![](https://imgur.com/NXL1Btu.png)

在每台節點上設定 Zookeeper 識別用唯一 id

```bash {linenostart=13}
echo "1" > /var/lib/zookeeper/myid
```

啟動 zookeeper, 並設為開機自動啟動

```bash {linenostart=14}
sudo systemctl start zookeeper
sudo systemctl enable zookeeper
```

下一篇將介紹 Kafka 叢集的安裝。

## source

[1] https://medium.com/@bikas.katwal10/why-zookeeper-needs-an-odd-number-of-nodes-bb8d6020e9e9
