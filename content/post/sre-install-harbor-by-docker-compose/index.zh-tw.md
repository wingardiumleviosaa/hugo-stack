---
title: 使用 docker-compose 建立 Harbor 私有倉庫 (w/ https)
tags:
  - Docker
categories:
  - SRE
date: 2022-04-10 15:55:00
slug: install-harbor-by-docker-compose
---

默認情況下，Harbor 不附帶證書，可以直接使用 http 訪問。但在正式上線的環境中，建議配置 https。

<!--more-->

## 準備 https 所需證書

在生產環境中，建議使用由受信任的第三方 CA 簽名的證書。在測試或開發環境中，則可以使用自簽的 CA。以下範例為使用 openssl 生成 CA，分為使用 DNS 或是直接使用 IP。

### 使用域名(DNS)

1. 準備 x509 v3 服務檔，假設 harbor 伺服器使用的 DNS 為 `harbor.ula.com`。

```bash
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor.ula.com
DNS.2=ula.com
DNS.3=harbor
EOF
```

2. 準備金鑰及證書的 shell script

```
vim prepare_ssl.sh
```

貼上以下內容

```bash
#!/bin/bash

CURDIR="`pwd`"/"`dirname $0`"
HOSTNAME=$1
DOMAIN=$2

INSTALL_PATH="/data/certs"
mkdir -p $INSTALL_PATH

cd $INSTALL_PATH
# generate CA certificate private key
openssl genrsa -out ca.key 4096

# generate CA certificate
openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=TW/ST=Taipei/L=Taipei/O=$HOSTNAME/OU=Personal/CN=$HOSTNAME.$DOMAIN" -key ca.key -out ca.crt

# generate server's private key
openssl genrsa -out $DOMAIN.key 4096

# Generate a certificate signing request (CSR)
openssl req -sha512 -new -days 3650 -subj "/C=TW/ST=Taipei/L=Taipei/O=$HOSTNAME/OU=Personal/CN=$HOSTNAME.$DOMAIN" -key $DOMAIN.key -out $DOMAIN.csr

# use v3.ext to generate certificate for harbor host
openssl x509 -req -sha512 -days 3650 -extfile $CURDIR/v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in $DOMAIN.csr -out $DOMAIN.crt

# Convert *.crt to *.cert, for use by Docker.
openssl x509 -inform PEM -in $DOMAIN.crt -out $DOMAIN.cert
chmod 400 $DOMAIN.key
```

3. 執行腳本創建相關證書及金鑰

```
./prepare_ssl.sh harbor ula.com
```

### 使用 IP

1. 準備 x509 v3 服務檔

```bash
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = IP:10.1.5.142
EOF
```

2. 準備金鑰及證書的 shell script

```
vim prepare_ssl.sh
```

貼上以下內容

```bash
#!/bin/bash

CURDIR="`pwd`"/"`dirname $0`"
IP=$1

INSTALL_PATH="/data/certs"
mkdir -p $INSTALL_PATH

cd $INSTALL_PATH
# generate CA certificate private key
openssl genrsa -out ca.key 4096

# generate CA certificate
openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=TW/ST=Taipei/L=Taipei/O=$IP/OU=Personal/CN=$IP" -key ca.key -out ca.crt

# generate server's private key
openssl genrsa -out $IP.key 4096

# Generate a certificate signing request (CSR)
openssl req -sha512 -new -days 3650 -subj "/C=TW/ST=Taipei/L=Taipei/O=$IP/OU=Personal/CN=$IP" -key $IP.key -out $IP.csr

# use v3.ext to generate certificate for harbor host
openssl x509 -req -sha512 -days 3650 -extfile $CURDIR/v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in $IP.csr -out $IP.crt

# Convert *.crt to *.cert, for use by Docker.
openssl x509 -inform PEM -in $IP.crt -out $IP.cert
chmod 400 $IP.key
```

3. 執行腳本創建相關證書及金鑰

```
./prepare_ssl.sh 10.1.5.142
```

因為本次建立沒有特別新增域名，所以以下範例是使用 IP 位址建立。

## 開始安裝

### 下載安裝檔

```
cd ~
wget https://github.com/goharbor/harbor/releases/download/v2.4.2/harbor-offline-installer-v2.4.2.tgz
tar zxvf harbor-offline-installer-v2.4.2.tgz
```

### 修改 harbor 配置檔

```
cd ~/harbor
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
```

針對以下三處修改

```yaml
# 沒有域名，寫 harbor 所在伺服器 ip
hostname: 10.1.5.142
http:
  # 因主機有其他服務已佔用 80 port 所以改成 8080
  port: 8080
https:
  # 因主機有其他服務已佔用 443 port 所以改成 4433
  port: 4433
  # 設定證書及金鑰路徑
  certificate: /data/certs/10.1.5.142.crt
  private_key: /data/certs/10.1.5.142.key

# harbor 預設登入密碼
harbor_admin_password: Harbor12345
```

### 安裝

使用安裝包中的 prepare 腳本生成配置

```
cd ~/harbor
./prepare
```

會產生以下結果

```
prepare base dir is set to /root/harbor
Unable to find image 'goharbor/prepare:v2.4.2' locally
v2.4.2: Pulling from goharbor/prepare
6c964f3b879f: Pull complete
3042f4284e2c: Pull complete
7a91ea24644b: Pull complete
b3c79e094a18: Pull complete
110443f7949d: Pull complete
1fbee02500d7: Pull complete
8675c3a61fe1: Pull complete
d48514bf9c2b: Pull complete
Digest: sha256:b14da6252672c596291f02a5e76fbb065f558aaf0a647f90e1c77aefaa29db74
Status: Downloaded newer image for goharbor/prepare:v2.4.2
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
```

使用安裝包中的腳本 install.sh 開始安裝 harbor

```
./install.sh
```

執行成功的話，會產生以下結果

```
[Step 0]: checking if docker is installed ...
Note: docker version: 20.10.14
[Step 1]: checking docker-compose is installed ...
Note: docker-compose version: 1.26.2
[Step 2]: loading Harbor images ...
[Step 3]: preparing environment ...
[Step 4]: preparing harbor configs ...
[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating redis         ... done
Creating registryctl   ... done
Creating registry      ... done
Creating harbor-db     ... done
Creating harbor-portal ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----
```

</br>

![](https://imgur.com/bg71xDV.png)

## 配置 docker 以登入私有倉庫

建立 docker 對應倉庫的證書存放目錄

```
mkdir -p /etc/docker/certs.d/10.1.5.142:4433
cp 10.1.5.142.key /etc/docker/certs.d/10.1.5.142:4433/
cp 10.1.5.142.cert /etc/docker/certs.d/10.1.5.142:4433/
cp ca.crt /etc/docker/certs.d/10.1.5.142:4433/
```

重啟 docker

```
systemctl restart docker
```

登入遠端倉庫

```
docker login 10.1.5.142:4433 -u admin -p Harbor12345
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

測試推送 imgage

```
docker pull busybox
docker tag busybox:latest 10.1.5.142:4433/test/busybox:latest
docker push 10.1.5.142:4433/test/busybox:latest
```

</br>

![](https://imgur.com/p2cdmoz.png)

![](https://imgur.com/98IsZit.png)

## Reference

- https://goharbor.io/docs/2.4.0/install-config/
