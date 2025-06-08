---
title: "[TCP/IP] 網路基礎概念- 傳紙條、TCP/IP 四層概述"
categories: ["SRE"]
tags: ["Network"]
date: 2020-06-30 23:04:00
slug: "tcpip-4layer"
---

## 網路

網路的目的就是為了溝通，可以把整個網路運作想像成傳紙條。

<!--more-->

### 一開始單純的傳紙條

只需：

- 寫明來源 (client IP)
- 寫明目的地 (server IP)
- 經過三次前置作業 (發送端發訊息、接收端接訊息並回復、發送端收到訊息)，確保雙方都能收發。(TCP 三次交握)
  ![](https://imgur.com/yK4lTIu.png)

### 把動作標準化

- 標準化內容格式，分為 header & body
  header：擴充性強，放額外資訊。
  body：放主要的訊息。
- 用狀態碼標準化 server 回應結果 (200, 301, 404, 502...)
- 用動詞標準化 client 要求的動作 (GET, POST, PUT, DELETE...)

### 進入規模化

愈來愈多的服務下，使用 **port** 將不同服務標示開來

- 一個 port 負責一個服務 (http:80, ftp:21...)
- 不同服務有不同訊息格式 (比如 GET 不須帶 body，POST 要帶 request body)
- 有些不需要使用者回傳訊息的服務，意即不需要經過三次確認 (UDP)

---

## protocol

協定就是一個標準，為了要讓彼此能夠溝通而建立的一個規範，有了標準就可以規模化讓它變成在不同作業系統、環境下的一個共同的準則。

---

## TCP/IP

網路的層級，理論上有標準的 OSI 七層。但實際上網路的實作，通常都以 **TCP/IP** 四層模型為主流，每一層都負責不同的事情、有不同的通訊協定。

![](https://imgur.com/9QQQFU0.png)
[1]

上層的協定都是建立在下層之上的，舉例 http 建立在 TCP 之上，TCP 又建立在 IP 之上...以此類推。

### TCP/ IP - 鏈結層 Link Layer

位於整個網路架構的最底層，負責制定資料傳輸的「實體」規格。所謂「實體」，就是我們眼睛看的到，手可以摸到的部分。以乙太網路來說，凡是接頭規格、傳輸線材種類、MAC 實體位址，都是鏈結層負責的內容。

{{< notice info >}}
剩下的三層，網路層、傳輸層、以及應用層將分為後面三篇文章紀錄。
{{< /notice >}}

## source

[1]http://linux.vbird.org/linux_server/0110network_basic.php  
[all] 記此篇為觀看 Lidemy NET101 的筆記，圖片來源以及部分內容取自上課影片
