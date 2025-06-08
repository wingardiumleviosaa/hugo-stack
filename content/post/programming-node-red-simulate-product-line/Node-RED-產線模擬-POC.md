---
title: "[Node-RED] Production Line Simulation"
tags:
  - Node-RED
categories:
  - Programming
date: 2020-11-03 16:00:00
slug: production-line-simulation-by-nodered
---

<!--more-->

## Production Line (gateway1)

### 初始流程

![](https://imgur.com/PYT3rrq.png)

首先第一個 inject flow (inject once at start) 會去下 http request 去跟事先寫好的 NodeJS express http server ([參考這篇](https://www.footmark.info/programming-language/nodejs/nodejs-restful-webapi-mysql/))索取事先放在 MySQL 的訂單資料 (目前僅設計為一筆)，function node 會放所有會使用到的 global 變數。

```js
global.set("wo", msg.payload[0].wo);
global.set("totalPcs", msg.payload[0].qty);
node.warn(global.get("wo"));
node.warn(global.get("totalPcs"));
global.set("startTime", 1603670400);

global.set("m1Status", "Running");
global.set("m1DonePcs", 0);
// node.warn(global.get('m1DonePcs'));

global.set("m2Status", "Stop");
global.set("m2DonePcs", 0);

global.set("m3Status", "Stop");
global.set("m3DonePcs", 0);

global.set("ng", 0);
global.set("ok", 0);

return msg;
```

### 送片流程

接下來的 `START!!` inject flow 會用 link node 形成一個大迴圈，去循迴產線。進入每一台機器前會有一個 switch node 判斷目前機器運行狀態是 `Running` 還是 `Idle`，在 `Running` 條件下，模擬機器製作一片的時間 `Producing...` 後，將每一台的完成片數加一，再將機器狀態設為閒置。接著再用 link node 連接回自己的 switch 判斷。

```js
global.set("m1DonePcs", global.get("m1DonePcs") + 1);
global.set("m1Status", "Idle");

node.warn("m1的第" + global.get("m1DonePcs") + "完成");
node.warn(global.get("m1Status"));
// node.warn 為了 debug 方便，將顯示於 debug tab

return msg;
```

在 `Idle` 條件下，讓機器休息三秒模擬過站，再把第二台機器喚醒，並準備送進下一片，再把自己的狀態改成運作。結束後 link 到第二台機器的 switch。

```js
global.set("m2Status", "Running");
node.warn("m1第" + global.get("m1DonePcs") + "片完成, 開始送到 m2");
// 機器一把自己的狀態改成運作
global.set("m1Status", "Running");

return msg;
```

</br>

{{< notice info >}}
由此可推第一片與下一片的時間差為 producing 時間加上過站三秒，因此 START!! inject node 的 intervel 時間設為上述時間差再加上三秒(送到第一台機器的過站時間)。
{{< /notice >}}

機器二為 `Running` 時完成片數加一、狀態變閒置；機器二做完後為 `Idle` 時，將機器三的狀態喚醒；

```js
global.set("m3Status", "Running");
node.warn("m2第" + global.get("m2DonePcs") + "片完成,開始送到 m3");
node.warn(global.get("m2Status"));
return msg;
```

機器三為 `Running` 時完成片數加一、狀態變閒置；機器三做完後為 `Idle` 時，**就結束這一片產品的流程了!** 這邊要特別想一下，不用再接回機器一，因為前面已經設了 intervel，第二片會自己送進來，且機器一的狀態是自己控制的，因為第一片完成後就進第二片了。

```js
node.warn("m3 第 " + global.get("m3DonePcs") + " 片完成");
return msg;
```

### 模擬資料流程

從第三個 flow 開始就是模擬產線上各台機器的 sensor 資料了。
第一台機器每秒更新

```js
var startTime = global.get("startTime");
msg.payload = {
  vaccum: {
    content: (Math.random() * 1.5).toFixed(1),
  },
  silverTarget: {
    content: Math.floor((new Date().getTime() / 1000 - startTime) / 60),
  },
  depositionspeed: {
    content: (1000 + Math.random()).toFixed(2),
  },
  m1Status: {
    content: global.get("m1Status"),
  },
  m1DonePcs: {
    content: global.get("m1DonePcs"),
  },
  wo: {
    content: global.get("wo"),
  },
  totalPcs: {
    content: " / " + global.get("totalPcs"),
  },
};
return msg;
```

datasource node 按照 msg.payload object 定義的 key 接收資料。

![](https://imgur.com/a6srloB.png)

第二台機器每秒更新

```js
var pressure, humidity, particle, m2Status, m2DonePcs;
pressure = Math.floor(Math.random() * 20 + 70).toFixed(0);
humidity = Math.floor(Math.random() * 40 + 30).toFixed(0);
particle = Math.floor(Math.random() * 50 + 60).toFixed(0);
msg.payload = {
  pressure: {
    content: pressure,
  },
  pressureAlarm: {
    turnOn: 1, // 0: Turn OFF, 1: Turn ON
    color: pressure >= 88 || pressure <= 73 ? 0 : 2, // color index
    mode: pressure >= 88 || pressure <= 73 >= 500 ? 1 : 0, // 0: means stable, 1: means blink
  },
  humidity: {
    content: humidity,
  },
  humidityAlarm: {
    turnOn: 1, // 0: Turn OFF, 1: Turn ON
    color: humidity >= 67 || humidity <= 35 ? 0 : 2, // color index
    mode: humidity >= 67 || humidity <= 35 ? 1 : 0, // 0: means stable, 1: means blink
  },
  particle: {
    content: particle,
  },
  particleAlarm: {
    turnOn: 1, // 0: Turn OFF, 1: Turn ON
    color: particle >= 105 ? 0 : 2, // color index
    mode: particle >= 105 ? 1 : 0, // 0: means stable, 1: means blink
  },
  m2Status: {
    content: global.get("m2Status"),
  },
  m2DonePcs: {
    content: global.get("m2DonePcs"),
  },
};
return msg;
```

第三台機器有兩個 flow，第一個 flow 每秒抓取機器狀態 m3Status 以及 m3DonePcs 完成數(代碼參考上兩台)，第二個 flow 用來模擬機器三檢測機的不良數，這邊的 intervel 也須留意設為一片的製作時間加上一兩秒，不能設為每秒抓取的原因是因為只要發現完成數除以某值整除的話就會有一片不良品，若在製作時間內都是整除該值的話，每秒抓取會每秒不良品加一，所以間格時間應該設為一片的製作時間。

```js
if (global.get("m3DonePcs") > 0 && global.get("m3DonePcs") % 5 === 0) {
  global.set("ng", global.get("ng") + 1);
}
global.set("ok", global.get("m3DonePcs") - global.get("ng"));
msg.payload = {
  ng: {
    content: global.get("ng"),
  },
  ok: {
    content: global.get("ok"),
  },
};
return msg;
```

### Dashboard 畫面

![](https://imgur.com/j1N5IbE.gif)

## Environment Monitor (gateway2)

![](https://imgur.com/1mz3O3j.png)

將使用 modsim32 ([下載](https://www.win-tech.com/html/demos.htm)、[啟用序號](https://www.findserialnumber.net/modsim-32-4-a00-04-serial-number-keygen-7d50fa31.html)) 模擬環境的四種 sensor 資料，再使用 modbus node 統一接收後拆開丟到對應的 datasource node。

![](https://imgur.com/kbDlXLP.png)

```js
var temperature = msg.payload.sensors.results[0];
var humidity = msg.payload.sensors.results[1];
var co2 = msg.payload.sensors.results[2];
var pm25 = msg.payload.sensors.results[3];
msg.payload = {
  temperature: {
    content: temperature,
  },
  humidity: {
    content: humidity,
  },
  co2: {
    content: co2,
  },
  pm25: {
    content: pm25,
  },
};
return msg;
```

## Edge Server

因為一鍵上雲的設計，從 gateway 部屬上 edge server 的 node 不能再次被 deploy 到其他伺服器，因為邊緣伺服器還打算部屬到雲端伺服器中，故這邊稍微做了點修改。

![](https://imgur.com/rDnCNiZ.png)

將從地端部屬上來的 flow 中的 datasource 複製到另一個自己的 flow 中，再宣告一個 global 變數接收上來的資料。收到的資料內容如下:

![](https://imgur.com/NzQ7BQw.png)

將字串變回 JSON 後再接到自己的 datasource 中，msg.payload 的資料格式依照使用的 chart 不同而不同。

```js
var obj = JSON.parse(global.get("gw1"));
//node.warn(Number(obj.temperature.content));
global.set("temp", Number(obj.temperature.content));
global.set("humi", Number(obj.humidity.content));
global.set("co", Number(obj.co2.content));
global.set("pm", Number(obj.pm25.content));
var now = new Date().getTime();
msg.payload = {
  tstamp: now,
  temperature: global.get("temp"),
  humidity: global.get("humi"),
  co2: global.get("co"),
  pm25: global.get("pm"),
};
return msg;
```

</br>

```js
var obj = JSON.parse(global.get("gw2"));
msg.payload = {
  m1Status: {
    content: obj.m1Status.content,
  },
  m1DonePcs: {
    content: obj.m1DonePcs.content,
  },
  wo: {
    content: obj.wo.content,
  },
  totalPcs: {
    content: obj.totalPcs.content,
  },
};
return msg;
```

dashboard 畫面如下:

![](https://imgur.com/fg9Yays.gif)
