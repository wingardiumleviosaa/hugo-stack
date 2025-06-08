---
title: "[SSL] 用 Let's Encrypt & Certbot 為網站加密"
categories:
  - SRE
tags: ["SSL"]
slug: "ssl-letsencrypt"
date: 2020-02-13 21:32:00
---

在講求資訊安全的時代，大部分的網站幾乎都使用 https 做為網站的通訊協定，這篇文章將記錄怎麼使用免費的第三方憑證 Let's Encrypt & Certbot，為網站添加安全保障。

<!--more-->

## Let's Encryt

Let’s Encrypt 是一個免費、自動化且開放的憑證機構 (Certificate Authority, CA)，取得憑證後，可為網站提供 SSL/TLS 加密。

Let’s Encrypt 使用 ACME 協定，來驗證申請網域控制權。使用者透過 ACME 客戶端軟體取得並管理憑證。最常見的 ACME 客戶端軟體為 <mark>Certbot</mark>

{{< notice note >}}
ACME (Automatic Certificate Management Environment): 自動憑證管理環境。由  Let's Encrypt 所實作的協議，與它相容的軟體可以透過此協議與 Let’s Encrypt 溝通以獲得憑證。[1]
{{< /notice >}}

## 實作

在 [Certbot](https://certbot.eff.org/instructions) 官方網站有快速指引，只需要選擇網站使用的 HTTP Server 以及作業系統，下方就會列出指令，可以直接複製並安裝。

![certbot](https://imgur.com/6g4Uh5t.png)

本文以 Ubuntu 18.04 + Nginx HTTP Server 實作。

### 1. 加入 Certbot PPA

{{< notice note >}}
PPA(Personal Pakage Archives)：個人軟體包文件，可加入個人開發者的  repository，使其他使用戶安裝和更新。
{{< /notice >}}

請依序在 console 中下以下 command

```bash
// 檢查更新
$ sudo apt-get update

// 安裝套件管理的套件
$ sudo apt-get install software-properties-common

// 啟用universe倉庫
$ sudo add-apt-repository universe

// 加入 certbot ppa repository
$ sudo add-apt-repository ppa:certbot/certbot

// 再更新一次套件資訊
$ sudo apt-get update

```

### 2. 安裝 Certbot

```
$ sudo apt-get install certbot python-certbot-nginx
```

### 3. 取得並安裝憑證

依使用者需求有下面幾種不同的做法

:pushpin: <span style="font-size:13.28 px">**全自動**</span>
讓 certbot 自動編輯完 nginx 配置文件

```
$ sudo certbot --nginx
```

:pushpin: <span style="font-size:13.28 px">**半自動**</span>
由使用者自己設置 nginx 配置文件

```
$ sudo certbot certonly --nginx
```

:pushpin: <span style="font-size:13.28 px">**webroot**</span>
若系統有不能中斷的需求，可以使用 `webroot` 套件，製作憑證過程不影響伺服器運作。  
安裝前需要先在 nginx 設定 acme-challenge，以證明具有網域控制權；
**a.**　設定 acme-challenge，進入/etc/nginx/site-available/編輯 default 文件

```
$ sudo vi /etc/nginx/site-available/default
```

**b.**　在 default 文件中，在 server block 中加入下面設定

```
server {
 #略
	location ~ /.well-known {
    	allow all;
	}
}
```

**c.**　執行 Certbot 取得憑證，其中 /var/www/html/ 為 Nginx 預設站點

```
$ sudo certbot certonly --webroot -w /var/www/html/ -d www.domain.com
```

上述任一方式，安裝過程都會需要輸入用於收通知的信箱。安裝完成後，憑證會放在`/etc/letsencrypt/live/www.domain.com`下，共四個檔案 `cert.pem`、`chain.pem`、`fullchain.pem`、`privkey.pem`。

### 4. 產生 Diffie-Hellman 密碼組合 (optional)

迪菲-赫爾曼密鑰交換[2] 是一種安全協定，可以讓雙方在沒有對方任何預先資訊的條件下通過不安全通訊建立一個金鑰。此金鑰可在後續的通訊中將作為對稱金鑰來加密通訊內容。  
在 `/etc/ssl/certs/` 下產生一個 2048bit 的 Diffie-Hellman 金鑰：

```bash
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

### 5. 設定 Nginx 配置文件

產生一個 nginx 的 configuration snippets 來配置 SSL 的相關設定，主要參考自 Cipherli.st 建議的 Nginx 設定：

```bash
$ sudo touch /etc/nginx/snippets/ssl-params.conf
$ sudo  vi /etc/nginx/snippets/ssl-params.conf
```

片段（Snippet）是一個編程用語，指的是原始碼、機器碼、文本中可重複使用的小區塊。[4]

插入以下：[3]

```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s; // Google DNS IP
resolver_timeout 5s;
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
ssl_dhparam /etc/ssl/certs/dhparam.pem; // step4 dh金鑰存放位址
```

接著依照需求，看是否需要保留 http ，或是僅保留 https 而 http 設定自動轉向到 https

```bash
$ sudo  vi /etc/nginx/site-enabled/default
```

```
server {
  listen 80 default_server;
  listen [::]:80 default_server;
  server_name domain.com www.domain.com;
  return 301 https://$server_name$request_uri;  // 加上這行讓 http port 轉向 https
}


server {
  listen [::]:443 ssl ipv6only=on http2 default_server;
  listen 443 ssl http2 default_server; # managed by Certbot
  server_name domain.com www.domain.com;
  ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;
  include snippets/ssl-params.conf;
  location ~ /.well-known {
    allow all;
  }
}

```

### 6. 測試並啟用 nginx

設定完成後便可以測試 nginx 配置是否正確

```bash
$ sudo nginx -t
```

如果有出現：

```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

就表示正常，便可重新啟用 nginx

```bash
$ sudo systemctl restart nginx
```

### 7. 設定排程自動更新憑證

因為 Let’s Encrypt 的每一個憑證期限為三個月，快過期的時候需要輸入指令 certbot renew 更新憑證，

加入參數  sudo certbot renew --dry-run ，能測試 Cerbot 是否能夠正常執行憑證更新。

但為了方便，以下將用 cron job 去自動更新憑證。

```bash
//確認certbox位址
$ which certbox

//開啟排程設定
sudo crontab -e
```

並加入以下排程：每個禮拜一的凌晨 2:00 進行憑證檢查及更新

```
00 2 * * 1 /usr/bin/certbot renew --quiet --post-hook "/etc/init.d/nginx restart"
```

</br>

```bash
//重啟crontab
sudo /etc/init.d/cron restart
```

## 參考資料

- [1] [letsencrypt](https://letsencrypt.org/zh-tw/docs/glossary/)
- [2] [Diffie–Hellman key exchange](https://zh.wikipedia.org/zh-tw/%E8%BF%AA%E8%8F%B2-%E8%B5%AB%E7%88%BE%E6%9B%BC%E5%AF%86%E9%91%B0%E4%BA%A4%E6%8F%9B)
- [3] [cipherlist](https://syslink.pl/cipherlist/)
- [4] [Snippet](https://zh.wikipedia.org/wiki/%E7%89%87%E6%AE%B5)
