---
title: 把 Router 當 Switch 用
categories: ["SRE"]
tags: ["Network"]
date: 2022-05-16 17:53:01
slug: using-router-as-switch
---

在公司架環境時，剛好遇到要讓幾台電腦都要直接連公司網段的情況，可是手上只有一台 router，如果把現有的 IP 網路線插在 WAN port 上，那麼插在 LAN port 的機器都會是虛擬網段，不符所需。後來 google 發現只要照下面二個步驟就可以讓 router 變身成 switch：

<!--more-->

1. 連進 router 的 web console，把 dhcp server 功能關掉
2. 把原先接在 WAN port 的網路線改接到 LAN port
   以手上這台 totolink ac5 為例，只要將電腦跟公司內部 router 出來的實體網路線都插到中間三個 LAN port 即可。

![](https://imgur.com/9tIpqnX.png)

補充一下基本知識

![](https://imgur.com/R2T6Rzc.png)

## Hub、Switch、Router 共同點在於

連結多台電腦、傳遞封包，延伸網路範圍、進行廣播。

## Hub

集線器用一個 port 直接連到網路上，大部分是有 DHCP server，其餘電腦接在這集線器上，就能自動拿到 IP。集線器所有 port 共享一個頻寬，同樣的封包會傳送至集線器所連結的每一台電腦，造成過多的廣播封包，影響網路傳輸的整體效能。

## L2 Switch

交換器是一種負責網路橋接（network bridging）的網路硬體設備，會讀取網路卡的 MAC 位址來轉發資料，將資料準確地送達目的地。交換器的每個 port 都享有一個專屬的頻寬並具備資料交換功能，使得網路傳輸效能得於同一時間內所能傳輸的資料量較大

## L3 Switch

如果再把路由表的功能加入 L2 Switch，那麼它就會變成 L3 Switch，可以為 VLAN 建立適當的路由表，讓效能更加提昇。L3 的交換器又稱為 IP Switch 或 Switch Router，透過專屬的 ASIC 晶片來解析第三層表頭（如 IP Header）以達到傳送目的，因此通常可以提高到每秒百萬封包的效能以及數十個高速乙太網路連接埠之容量。L3 Switch 的路由表可以對 VLAN 做更有效的管制，讓廣播封包不會無限制的傳送。

L3 Swtich 比較專注在企業內網的 LAN 環境應用，通常會有提供大量的 Port 數，對於封包轉送的效能高。

## Router

Router 通常定位在跨 WAN 的邊界連接用，可區隔不同的 IP 網段，可拿來當作 NAT、防火牆、負載平衡或 VoIP 等服務。通常不會有太多 Port (因為很少會有數十條 WAN 同時接進來)。雖軟體速度較硬體慢 (加上在網路協定中，越往上跑，封包要拆開越多層)，但在實際運用上較有彈性。
