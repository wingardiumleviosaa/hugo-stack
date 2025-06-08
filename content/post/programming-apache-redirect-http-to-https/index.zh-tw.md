---
title: Apache Redirect http to https
tags:
  - PHP
categories:
  - Programming
date: 2022-05-15 09:40:00
slug: redirect-http-to-https-on-apache
---

目前手上有一個使用 php 開發的網頁，web server 使用 Apache (httpd) 實做。本篇記錄如何將 http 自動轉址到 https。

<!--more-->

## mod_rewrite.c

編輯 httpd config 檔，`/etc/httpd/conf/httpd.conf`，在 `<VirtualHost *:80>` 的區塊下新增以下 rewirte module：

```
<IfModule mod_rewrite.c>
  RewriteEngine on
  RewriteRule ^ - [E=protossl]

  RewriteCond %{HTTPS} on
  RewriteRule ^ - [E=protossl:s]

  RewriteCond %{HTTPS} !=on
  RewriteRule ^ https://www.ula.com/$1 [L,R=301]
</IfModule>
```

作業環境為 CentOS，重啟 httpd

```
sudo systemctl restart httpd
```

## Reference

- https://stackoverflow.com/questions/57341254/how-to-redirect-http-to-https-on-apache
