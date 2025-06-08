---
title: "[Docker] 使用 docker-compose 建立 GitLab (w/ https)"
tags:
  - Docker
  - Git
categories:
  - DevOps
date: 2021-10-29 13:48:00
slug: install-gitlab-server-by-docker-compose
---

## 安裝 docker & docker compose

請參考之前的筆記

- [Install Docker & Docker Compose on CentOS](https://ulahsieh.netlify.app/p/install-docker-and-docker-compose-on-centos7/)
- [Install Docker & Docker Compose on Ubuntu18.04](https://ulahsieh.netlify.app/p/install-docker-and-docker-compose-on-ubuntu18/)

<!--more-->

## 準備自簽憑證

### 建立 ssl.conf 設定檔

```
[req]
prompt = no
default_md = sha256
default_bits = 2048
distinguished_name = dn
x509_extensions = v3_req

[dn]
C = TW
ST = Taiwan
L = Taipei
O = ABC Inc.
OU = IT Department
emailAddress = ulahsieh@abc.com
CN = 10.1.5.8

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = 10.1.5.8
IP.1 = 10.1.5.8
```

`[dn]`  區段 ([Distinguished Name](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)) 為憑證的相關資訊

- C: 國碼，臺灣是 TW
- ST: 州
- L: 地區
- O: 組織名稱
- OU: 部門名稱
- emailAddress: E-Mail
- CN: 憑證名稱，通常填域名名稱

`alt_names`  用來設定 SSL 憑證的域名。可以設定很多組，也可以把區網的 IP 填上去。

### 產生自簽憑證與私密金鑰

```bash
openssl req -x509 -new -nodes -sha256 -utf8 -days 3650 -newkey rsa:2048 -keyout gitlab.key -out gitlab.crt -config ssl.conf
```

產生 stronger DHE parameters 加強 server 安全性

```bash
openssl dhparam -out dhparam.pem 2048
```

完成後，總共會產生出兩個檔案，分別是：

1. `server.key` (私密金鑰) (使用 PEM 格式) (無密碼保護)
2. `server.crt` (憑證檔案) (使用 PEM 格式)
3. `dhparam.pem`

### 將憑證移到指定要掛載的 volume 路徑下的 certs 資料夾下

```bash
mkdir -p /srv/docker/gitlab/gitlab/certs
cp gitlab.key /srv/docker/gitlab/gitlab/certs/
cp gitlab.crt /srv/docker/gitlab/gitlab/certs/
cp dhparam.pem /srv/docker/gitlab/gitlab/certs/
chmod 400 /srv/docker/gitlab/gitlab/certs/gitlab.key # 將金鑰設為唯讀確保安全性
```

## 開始安裝 GitLab

### 下載 docker-compose.yaml 檔案

```bash
wget https://raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml
```

### 修改設定

請注意下方 gitlab 的 volumes 設定，因為 gitlab container 內部會預設到 `~/data/certs` 下找憑證 ，故使用指定的 bind mount path `/srv/docker/gitlab/gitlab`將前面放到 `certs` 目錄下的金鑰與證書 mount 進 container。

另外此 image 也提供三個參數指定金鑰與證書位置：

- SSL_KEY_PATH
- SSL_CERTIFICATE_PATH
- SSL_DHPARAM_PATH

```yaml
version: "2.3"

services:
  redis:
    restart: always
    image: redis:6.2
    command:
      - --loglevel warning
    volumes:
      - /srv/docker/gitlab/redis:/data:Z # 前面可以指定本機端要 mount 的路徑, 或取一個 volume 名字

  postgresql:
    restart: always
    image: sameersbn/postgresql:12-20200524
    volumes:
      - /srv/docker/gitlab/postgresql:/var/lib/postgresql:Z # 前面可以指定本機端要 mount 的路徑, 或取一個 volume 名字
    environment:
      - DB_USER=gitlab
      - DB_PASS=password
      - DB_NAME=gitlabhq_production
      - DB_EXTENSION=pg_trgm,btree_gist

  gitlab:
    restart: always
    image: sameersbn/gitlab:14.3.3
    depends_on:
      - redis
      - postgresql
    ports:
      - "80:80" ## 冒號前面的端口改為本機想要使用的端口
      - "10022:22"
      - "443:443" ## 如果要使用 https, container 端口要加上 443, 對應出冒號前面想要的本機端口
    volumes:
      - /srv/docker/gitlab/gitlab:/home/git/data:Z # 前面可以指定本機端要 mount 的路徑, 或取一個 volume 名字
    healthcheck:
      test: ["CMD", "/usr/local/sbin/healthcheck"]
      interval: 5m
      timeout: 10s
      retries: 3
      start_period: 5m
    environment:
      - DEBUG=false

      - DB_ADAPTER=postgresql
      - DB_HOST=postgresql
      - DB_PORT=5432

      - DB_HOST=postgresql
      - DB_PORT=5432
      - DB_USER=gitlab
      - DB_PASS=password
      - DB_NAME=gitlabhq_production

      - REDIS_HOST=redis
      - REDIS_PORT=6379

      - TZ=Asia/Taipei # 指定時區
      - GITLAB_TIMEZONE=Taipei # 指定時區

      - GITLAB_HTTPS=true # 如果需要使用 HTTPS，需要設為 true
      - SSL_SELF_SIGNED=true # 如果使用自簽証書，需要設為 ture

      - GITLAB_HOST=10.1.5.8 # 改為自己的域名, 或是 IP Address
      - GITLAB_PORT=443 # 若有啟用 HTTPS 則設為 HTTPS 本機使用的端口
      - GITLAB_SSH_PORT=10022
      - GITLAB_RELATIVE_URL_ROOT=
      - GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alphanumeric-string
      - GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alphanumeric-string
      - GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alphanumeric-string

      # 設定 root email & password, 用於第一次以 administrator 登入
      - GITLAB_ROOT_PASSWORD=nexdata.com # 請注意密碼需要至少八碼
      - GITLAB_ROOT_EMAIL=admin@nexdata.com

      - GITLAB_NOTIFY_ON_BROKEN_BUILDS=true
      - GITLAB_NOTIFY_PUSHER=false

      - GITLAB_EMAIL=notifications@example.com
      - GITLAB_EMAIL_REPLY_TO=noreply@example.com
      - GITLAB_INCOMING_EMAIL_ADDRESS=reply@example.com

      - GITLAB_BACKUP_SCHEDULE=daily
      - GITLAB_BACKUP_TIME=01:00

      # SMTP 用於發送郵件 (忘記密碼、通知等)
      - SMTP_ENABLED=false
      - SMTP_DOMAIN=www.example.com
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USER=mailer@example.com
      - SMTP_PASS=password
      - SMTP_STARTTLS=true
      - SMTP_AUTHENTICATION=login

      # IMAP 用於接收郵件
      - IMAP_ENABLED=false
      - IMAP_HOST=imap.gmail.com
      - IMAP_PORT=993
      - IMAP_USER=mailer@example.com
      - IMAP_PASS=password
      - IMAP_SSL=true
      - IMAP_STARTTLS=false

      # 下面是各種用於通過 GITHUB 等平台授權登錄設定
      - OAUTH_ENABLED=false
      - OAUTH_AUTO_SIGN_IN_WITH_PROVIDER=
      - OAUTH_ALLOW_SSO=
      - OAUTH_BLOCK_AUTO_CREATED_USERS=true
      - OAUTH_AUTO_LINK_LDAP_USER=false
      - OAUTH_AUTO_LINK_SAML_USER=false
      - OAUTH_EXTERNAL_PROVIDERS=

      - OAUTH_CAS3_LABEL=cas3
      - OAUTH_CAS3_SERVER=
      - OAUTH_CAS3_DISABLE_SSL_VERIFICATION=false
      - OAUTH_CAS3_LOGIN_URL=/cas/login
      - OAUTH_CAS3_VALIDATE_URL=/cas/p3/serviceValidate
      - OAUTH_CAS3_LOGOUT_URL=/cas/logout

      - OAUTH_GOOGLE_API_KEY=
      - OAUTH_GOOGLE_APP_SECRET=
      - OAUTH_GOOGLE_RESTRICT_DOMAIN=

      - OAUTH_FACEBOOK_API_KEY=
      - OAUTH_FACEBOOK_APP_SECRET=

      - OAUTH_TWITTER_API_KEY=
      - OAUTH_TWITTER_APP_SECRET=

      - OAUTH_GITHUB_API_KEY=
      - OAUTH_GITHUB_APP_SECRET=
      - OAUTH_GITHUB_URL=
      - OAUTH_GITHUB_VERIFY_SSL=

      - OAUTH_GITLAB_API_KEY=
      - OAUTH_GITLAB_APP_SECRET=

      - OAUTH_BITBUCKET_API_KEY=
      - OAUTH_BITBUCKET_APP_SECRET=
      - OAUTH_BITBUCKET_URL=

      - OAUTH_SAML_ASSERTION_CONSUMER_SERVICE_URL=
      - OAUTH_SAML_IDP_CERT_FINGERPRINT=
      - OAUTH_SAML_IDP_SSO_TARGET_URL=
      - OAUTH_SAML_ISSUER=
      - OAUTH_SAML_LABEL="Our SAML Provider"
      - OAUTH_SAML_NAME_IDENTIFIER_FORMAT=urn:oasis:names:tc:SAML:2.0:nameid-format:transient
      - OAUTH_SAML_GROUPS_ATTRIBUTE=
      - OAUTH_SAML_EXTERNAL_GROUPS=
      - OAUTH_SAML_ATTRIBUTE_STATEMENTS_EMAIL=
      - OAUTH_SAML_ATTRIBUTE_STATEMENTS_NAME=
      - OAUTH_SAML_ATTRIBUTE_STATEMENTS_USERNAME=
      - OAUTH_SAML_ATTRIBUTE_STATEMENTS_FIRST_NAME=
      - OAUTH_SAML_ATTRIBUTE_STATEMENTS_LAST_NAME=

      - OAUTH_CROWD_SERVER_URL=
      - OAUTH_CROWD_APP_NAME=
      - OAUTH_CROWD_APP_PASSWORD=

      - OAUTH_AUTH0_CLIENT_ID=
      - OAUTH_AUTH0_CLIENT_SECRET=
      - OAUTH_AUTH0_DOMAIN=
      - OAUTH_AUTH0_SCOPE=

      - OAUTH_AZURE_API_KEY=
      - OAUTH_AZURE_API_SECRET=
      - OAUTH_AZURE_TENANT_ID=
```

### 啟動 docker-compose

```bash
docker-compose up -d
```

啟動後可以進入 logs 看使否有成功跑起服務，建立過程大概要等三到五分鐘左右

```bash
docker-compose logs -f gitlab
```

![](https://imgur.com/ET4o1pg.png)

![](https://imgur.com/lRy0kBG.png)

完成後可以打開瀏覽器前往 GitLab，若有開啟 https 的設定，則瀏覽器會自動跳轉 redirect 到 https，使用在 yaml 檔設定的 root email & password 登入。

![](https://imgur.com/d9vuES3.png)

![](https://imgur.com/sRn09l0.png)

## Reference

- [sameersbn github & installation of the ssl certificates](https://github.com/sameersbn/docker-gitlab#installation-of-the-ssl-certificates)
- [https://ithelp.ithome.com.tw/articles/10204317](https://ithelp.ithome.com.tw/articles/10204317)
- [如何使用 OpenSSL 建立開發測試用途的自簽憑證 (Self-Signed Certificate)
  ](https://blog.miniasp.com/post/2019/02/25/Creating-Self-signed-Certificate-using-OpenSSL)
