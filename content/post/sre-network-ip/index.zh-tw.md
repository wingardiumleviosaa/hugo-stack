---
title: "[TCP/IP] 網路層 - IP"
description: "網路層中的 IP 概念說明。"
categories: ["SRE"]
tags: ["Network"]
date: 2020-06-30 23:20:00
slug: "tcpip-ip"
---

## 網路層 Network Layer

### IP

Internet Protocol，網絡上每一個節點都有一個獨立的 IP Address，當裝置連接網路時會被分配 IP，用以標識。通過 IP 位址，裝置間可以互相知道對方並通訊。  
有 IPv4 及 IPv6 兩個版本，IPv6 是為了要解決 IPv4 不足的問題。

<!--more-->

#### 固定 IP 　 v.s. 浮動 IP

固定 IP 指使用的 IP 都是同一個，不會因為斷線再重新連線而改變，適合給線上服務的伺服器使用．便於提供一些固定連線的服務(Web Server, Domain Name Server, Mail Server...)；  
浮動 IP 指每次連接時 IP 都會不同，通常給對 IP 位址不要求的一般用戶使用。且因為 IP 每次變動，比較不容易被駭客鎖定，相較於固定 IP 安全。

#### 實體 IP v.s. 虛擬 IP

實體 IP 又稱 Public IP，指的是可以在網路上溝通的 IP。在網路的世界裡，用實體 IP 辨識每一部電腦的位置。
虛擬 IP 又稱 Private IP，被設計用來解決實體 IP 不足的問題。透過 IP 分享器，讓多台電腦各自擁有虛擬 IP，並對外共用一個實體 IP。
預留的三個虛擬 IP 網段：

- A Class：10.0.0.0 - 10.255.255.255
- B Class：172.16.0.0 - 172.31.255.255
- C Class：192.168.0.0 - 192.168.255.255

![](https://imgur.com/gRSIi6a.png) [all]

#### NAT

擁有私有 IP 的內部使用者到要到網際網路需經過網路位址轉換(Network address translation, NAT)的機制，這個機制會把網路封包內容的虛擬 IP 變成真實 IP。

![](https://imgur.com/NvK2KsN.png) [1]

#### DNS

Domain Name Server，處理 Domain Name 與實際 IP 的轉換。

![](https://imgur.com/X9ulQwP.png)
[2]

## Reference

[1][configuring-a-nat-device-when-external-users-want-to-access-an-internal-server](https://support.huawei.com/enterprise/en/doc/EDOC1100027155?section=j00e&topicName=configuring-a-nat-device-when-external-users-want-to-access-an-internal-server)  
[2][best-fastest-bsnl-dns-servers/](https://techthatmatter.com/best-fastest-bsnl-dns-servers/)  
[all] 記此篇為觀看 Lidemy NET101 的筆記，圖片來源以及部分內容取自上課影片
