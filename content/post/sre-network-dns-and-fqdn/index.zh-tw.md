---
title: "DNS and FQDN"
tags:
  - Network
categories:
  - SRE
date: 2023-07-23T14:26:00+08:00
---

## FQDN

Fully Qualified Domain Name，是網際網路上特定計算機或主機的完整域名。

<!--more-->

## DNS

當我們申請域名時，會看到網址註冊服務商詢問是否有 DNS 的服務(大部分的網域提供商都會提供免費的 DNS 服務，如 GoDaddy, Cloudflare, Amazon Route 53)。DNS (Domain Name System) 是一種用於將域名轉換為 IP 地址的系統，以便在 Internet 上定位網絡資源。

## 紀錄類型

紀錄類型是為了輔助 DNS server (DNS 伺服器)可以更有效率地查找網址對應的 IP 位址，常使用到的紀錄類型如下:

### A (Address) Record:

A 紀錄代表的就是 IPv4 地址，是最常用到的紀錄類型。直接將域名解析為相應的 IP 地址，如將 "example.com" 解析為 "192.0.2.1"。

### AAAA (IPv6 Address) Record:

AAAA 記錄類似於 A 記錄，將域名映射到 IPv6 地址。若網站所在的主機是使用 IPv6 格式的 IP，則可選擇 AAAA 紀錄做為紀錄類型。

### CNAME 記錄

CNAME 記錄用於創建別名，將一個域名映射到另一個域名，讓 DNS 將網址解析到另一個目標網址上。如將 "www.example.com" 用 CNAME 記錄解析至 "example.com" 上。DNS 查詢步驟如下:

1. 網站訪客在輸入 "www.example.com" 時，DNS 會發現它是一個 CNAME 記錄，即 "example.com"。
2. 查詢 "example.com" 所設定 A 紀錄值中的主機 IP
   日後 "example.com" 設定的 A 紀錄值有所變更時，就不需要再去更改 "www.example.com" 紀錄值。因此在使用 CNAME 紀錄時，對應到的目標網址必須帶有 A 紀錄。

### MX (Mail Exchange) Record:

MX 記錄指定郵件伺服器的地址，該地址負責接收特定域名的電子郵件。例如 A 要寄 Email 給 B 時，B 設定的 MX 紀錄值可以讓 A 的郵件主機透過 MX 紀錄找到 B 負責收信的郵件主機，讓 A 可以順利將 Email 寄給 B。

### NS (Name Server) Record:

NS 記錄指定維護該域名 DNS 資訊的 DNS 伺服器，通常由域名註冊商設定，NS 記錄告訴 DNS 查詢該域名的資訊在哪裡可以找到。
名稱伺服器 (Name Server) 是一種 DNS 伺服器，上面儲存了網域的所有 DNS 記錄，包括 A 記錄、MX 記錄或 CNAME 記錄。
