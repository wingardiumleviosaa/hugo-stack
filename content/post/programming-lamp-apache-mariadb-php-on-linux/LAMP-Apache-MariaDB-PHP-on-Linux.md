---
title: 佈署 LAMP (Apache + MariaDB + PHP Web Server on Linux)
tags:
  - PHP
categories:
  - Programming
date: 2022-02-22 20:01:00
slug: deploy-lamp-on-linux
---

## 系統環境

- CentOS 7.9
- PHP 7.2.34
- Apache/2.4.6

```
[root@ewrula ~]# httpd -v
Server version: Apache/2.4.6 (CentOS)
Server built:   Jan 25 2022 14:08:43
```

- MariaDB 5.5.68

```
[root@ewrula ~]# mysql -V
mysql Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1
```

<!--more-->

## 安裝

### Apache

```bash
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```

安裝完畢並啟動後 Apache 已經可以存取了

![](https://imgur.com/I2Y1EXP.png)

### MariaDB

```bash
yum install mariadb-server mariadb
systemctl start mariadb
systemctl enable mariadb
```

設定 MariaDB 的 root 密碼：

```bash
/usr/bin/mysql_secure_installation
```

會跳出提示，預設是空密碼，故請按照指示 enter for none

```bash
Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorization.

New password: password
Re-enter new password: password
Password updated successfully!
Reloading privilege tables..
 ... Success!
```

測試連線 MariaDB：

```bash
mysql -u root -p
```

### php 7.2

目前 php 最高的穩定版本是 7.2，若直接採用 CentOS 中的 yum 安裝 `yum -y install php`，版本是 5.4，因此需要手動更新 rpm。

```bash
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

7.2 版本名為 72w，除了 php 庫外還需安裝其他常用套件：

```bash
yum -y install php72w php72w-bcmath php72w-cli php72w-common php72w-dba php72w-devel php72w-embedded php72w-enchant php72w-fpm php72w-gd php72w-imap php72w-interbase php72w-intl php72w-ldap php72w-mbstring php72w-mcrypt php72w-mysqlnd php72w-odbc php72w-opcache php72w-pdo php72w-pdo_dblib php72w-pear php72w-pecl-apcu php72w-pecl-apcu-devel php72w-pecl-imagick php72w-pecl-imagick-devel php72w-pecl-xdebug php72w-pgsql php72w-phpdbg php72w-process php72w-pspell php72w-recode php72w-snmp php72w-soap php72w-tidy php72w-xml php72w-xmlrpc
```

查看已安裝套件:

```
[root@ewrula ~]# php -m
[PHP Modules]
apcu
bcmath
bz2
calendar
Core
ctype
curl
date
dba
dom
enchant
exif
fileinfo
filter
ftp
gd
gettext
gmp
hash
iconv
imagick
imap
interbase
intl
json
ldap
libxml
mbstring
mysqli
mysqlnd
odbc
openssl
pcntl
pcre
PDO
pdo_dblib
PDO_Firebird
pdo_mysql
PDO_ODBC
pdo_pgsql
pdo_sqlite
pgsql
Phar
posix
pspell
readline
recode
Reflection
session
shmop
SimpleXML
snmp
soap
sockets
SPL
sqlite3
standard
sysvmsg
sysvsem
sysvshm
tidy
tokenizer
wddx
xdebug
xml
xmlreader
xmlrpc
xmlwriter
xsl
Zend OPcache
zip
zlib

[Zend Modules]
Xdebug
Zend OPcache

[root@ewrula ~]#
```

## 配置

### 設定防火牆

```bash
firewall-cmd --zone=public --permanent --add-port=80/tcp
firewall-cmd --zone=public --permanent --add-port=443/tcp
service firewalld restart
```

### 設定 apache conf 檔

```bash
vim /etc/httpd/conf/httpd.conf
```

主要要設定網頁路徑所在，詳細的設定項說明可以參考鳥哥的[文章](https://linux.vbird.org/linux_server/redhat6.1/linux_26wwwapache.php)

### 權限設定

```bash
chown -R apache.apache /var/www/blog
chmod -R 755 /var/www/blog
sudo chcon -t httpd_sys_rw_content_t /var/www/blog -R
```

### 重啟 server

```bash
systemctl restart httpd
```

## 解決 QueryException (2002) SQLSTATE[HY000] [2002] Permission denied 的錯誤

這是 SELinux 對 httpd 運行服務的安全控管機制，請先查詢 httpd 運行時的設定

```bash
getsebool -a | grep httpd
```

如果 httpd_can_network_connect_db 的設定是 Off (關閉)，請透過下列指令予以開啟

```bash
setsebool -P httpd_can_network_connect_db 1
```

執行過程：

```bash
[root@ewrula ewr]# getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off
httpd_can_network_memcache --> off
httpd_can_network_relay --> off
httpd_can_sendmail --> off
httpd_dbus_avahi --> off
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on
httpd_enable_ftp_server --> off
httpd_enable_homedirs --> off
httpd_execmem --> off
httpd_graceful_shutdown --> on
httpd_manage_ipa --> off
httpd_mod_auth_ntlm_winbind --> off
httpd_mod_auth_pam --> off
httpd_read_user_content --> off
httpd_run_ipa --> off
httpd_run_preupgrade --> off
httpd_run_stickshift --> off
httpd_serve_cobbler_files --> off
httpd_setrlimit --> off
httpd_ssi_exec --> off
httpd_sys_script_anon_write --> off
httpd_tmp_exec --> off
httpd_tty_comm --> off
httpd_unified --> off
httpd_use_cifs --> off
httpd_use_fusefs --> off
httpd_use_gpg --> off
httpd_use_nfs --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
[root@ewrula ewr]# setsebool -P httpd_can_network_connect_db 1
[root@ewrula ewr]#
[root@ewrula ewr]#
[root@ewrula ewr]#
[root@ewrula ewr]# getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> on
httpd_can_network_memcache --> off
httpd_can_network_relay --> off
httpd_can_sendmail --> off
httpd_dbus_avahi --> off
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on
httpd_enable_ftp_server --> off
httpd_enable_homedirs --> off
httpd_execmem --> off
httpd_graceful_shutdown --> on
httpd_manage_ipa --> off
httpd_mod_auth_ntlm_winbind --> off
httpd_mod_auth_pam --> off
httpd_read_user_content --> off
httpd_run_ipa --> off
httpd_run_preupgrade --> off
httpd_run_stickshift --> off
httpd_serve_cobbler_files --> off
httpd_setrlimit --> off
httpd_ssi_exec --> off
httpd_sys_script_anon_write --> off
httpd_tmp_exec --> off
httpd_tty_comm --> off
httpd_unified --> off
httpd_use_cifs --> off
httpd_use_fusefs --> off
httpd_use_gpg --> off
httpd_use_nfs --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
[root@ewrula ewr]# systemctl restart httpd
```

## Reference

- https://ithelp.ithome.com.tw/questions/10186688
