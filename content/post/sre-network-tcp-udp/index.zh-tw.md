---
title: "[TCP/IP] 傳輸層 - TCP , UDP"
author: Ula Hsieh
tags:
  - Network
categories:
  - SRE
date: 2020-06-30 23:30:00
---

## 傳輸層 Transport Layer

### Port

連接埠/端口。有了網際網路層的 IP 位址之後，就可以把檔案傳輸到目標電腦上，但是一台電腦上可能有許多不同的應用程式，沒辦法知道這個檔案要由電腦上的哪一個應用程式處理，因此出現了 port。使用 port 就可以區別這些服務，如果有服務的程式在監聽這個埠號的話，就會收到 request。

<!--more-->

常用的 port 有： http 80, https 443, ftp 21。

### TCP

Transmission Control Protocol 傳輸控制協定，TCP 為**可靠**的傳輸協定，通訊前須先建立「連線」，連線完成後進行「通訊」，通訊完畢後「中斷連線」，應用如 http, ftp。

![](https://imgur.com/Lk5Jcq1.png)

#### 建立連線 - 三次交握

1. Client 傳送建立連線訊息給 Server，裡面有個包個數字 x (例如 1000)。
2. Server 收到後，將這個 x 記錄下來，然後回發個訊息 y (例如 8000) 以及 ack(x+1)， x + 1 的目的是因為這樣才能證明是 Server 是收到 Client 的訊息)
3. Client 收到後，也回應個 ack(y+1)（同理這樣才能證明是 Server 寫的）
   經過以上三次交握，連線建立。

#### 傳送訊息

1. Client 發送資料長度為 20 byte 的資料給 Server
2. Server 收到後回 ack 為 x + 20 的訊息給 Client
3. Client 再次發送資料長度為 20 byte 的資料給 Server
4. 過一段時間 timeout 後，Client 再發送一次
5. Server 收到後回 ack 為 x + 20 的訊息給 Client

#### 結束連線 - 四次揮手

1. Client 發送一個斷線訊息 Finish & x 給 Server
2. Server 收到後回發 x + 1 訊息給 Client
3. 接下來 Server 接續發送一個斷線訊息 Finish & y 給 A
4. Client 收到後再回發一個 y + 1 訊息給 Server (Server 就正式斷線)
5. Client 正式斷線，資源釋放

### UDP

User Datagram Protocol，使用者資料元協定，UDP 為**不可靠**的傳輸協定，使用在要求速度的應用上，不在乎是否送達，實例如視訊，掉了一兩個畫面不影響功能。

## Reference

- https://ithelp.ithome.com.tw/articles/10205476
