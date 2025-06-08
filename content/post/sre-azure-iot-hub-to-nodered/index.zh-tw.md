---
title: Azure IoT Hub node-red 實做
tags:
  - Node-RED
categories:
  - DataEngineering
date: 2022-05-30 10:07:00
slug: azure-iot-hub-to-nodered
---

本篇記錄如何使用 Node-Red 實作 Azure IoT Hub 的資料傳輸。

<!--more-->

## 安裝 Node-red

在雲端以及地端兩邊都準備好 node-red

```
# 安裝 nodejs & npm
sudo apt update
sudo apt install nodejs -y
sudo apt install npm -y
sudo npm install npm@6.14.0 -g
sudo npm cache clean -f
sudo npm install -g n
sudo n 10.22.0 stable

# 安裝 node-red
sudo npm install -g --unsafe-perm node-red

# 啟動 node-red
node-red
```

## 建立 IoT Hub

在 azure 建立一個 IoT Hub

![](https://imgur.com/ioBi49C.png)

![](https://imgur.com/3o16CS1.png)

## 安裝 Node-Red Node

在雲端以及地端的機器中的 manage palette 安裝 node-red-contrib-azure-iot-hub

![](https://imgur.com/vzQ65M2.png)

![](https://imgur.com/F8foKFR.png)

## 準備 Local Node-red 的 workflow

### 註冊 Device

需要將機器註冊到 Azure IoT Hub 中，準備了下面的 workflow

![](https://imgur.com/8MFPMbM.png)

```json
[
  {
    "id": "8420253b.d369c8",
    "type": "tab",
    "label": "Flow 1",
    "disabled": false,
    "info": ""
  },
  {
    "id": "a937e034.afa3c",
    "type": "azureiothubregistry",
    "z": "8420253b.d369c8",
    "name": "Azure IoT Hub Registry",
    "x": 410,
    "y": 180,
    "wires": [["b7a4e98a.e50b18"]]
  },
  {
    "id": "dbdd68fc.579708",
    "type": "inject",
    "z": "8420253b.d369c8",
    "name": "Register Payload",
    "props": [{ "p": "payload" }],
    "repeat": "",
    "crontab": "",
    "once": false,
    "onceDelay": "",
    "topic": "",
    "payload": "{\"deviceId\":\"hello\"}",
    "payloadType": "json",
    "x": 180,
    "y": 180,
    "wires": [["a937e034.afa3c"]]
  },
  {
    "id": "b7a4e98a.e50b18",
    "type": "debug",
    "z": "8420253b.d369c8",
    "name": "Log",
    "active": true,
    "console": "false",
    "complete": "true",
    "x": 650,
    "y": 180,
    "wires": []
  }
]
```

在 Azure IoT Hub Registry node 中貼上 IoT Hub 的 connectionString (primary 或 secondary 皆可行)

![](https://imgur.com/JlPVBPT.png)

![](https://imgur.com/zuatlpk.png)

在 inject node 中設定 device id

![](https://imgur.com/9C53Qoa.png)

Deploy 後按下 inject，便註冊 device 成功了，可以看到在 debug window 中回傳了該 device 的金鑰。

![](https://imgur.com/Tg5Noge.png)

### 發送資料

註冊完機器後，開始準備丟資料，使用以下的 workflow

![](https://imgur.com/OjqPc7X.png)

```json
[
  {
    "id": "7db92193.aed53",
    "type": "tab",
    "label": "Flow 2",
    "disabled": false,
    "info": ""
  },
  {
    "id": "897ffdd4.12ba3",
    "type": "debug",
    "z": "7db92193.aed53",
    "name": "Log",
    "active": true,
    "console": "false",
    "complete": "true",
    "x": 870,
    "y": 200,
    "wires": []
  },
  {
    "id": "66c43e6.4f5f7c",
    "type": "azureiothub",
    "z": "7db92193.aed53",
    "name": "Azure IoT Hub",
    "protocol": "http",
    "x": 660,
    "y": 200,
    "wires": [["897ffdd4.12ba3"]]
  },
  {
    "id": "c5686695.31d5e8",
    "type": "inject",
    "z": "7db92193.aed53",
    "name": "Send Payload",
    "props": [],
    "repeat": "5",
    "crontab": "",
    "once": false,
    "onceDelay": "",
    "topic": "",
    "x": 200,
    "y": 200,
    "wires": [["84141515.9a1188"]]
  },
  {
    "id": "84141515.9a1188",
    "type": "function",
    "z": "7db92193.aed53",
    "name": "",
    "func": "let a = Math.floor(Math.random()*100)+1;\nmsg.payload={\n    \"deviceId\":\"device1\",\n    \"key\":\"pOX7pNKnt2aJoTy5JX4BYXidsezO+fr1sEz0TQQt1YM=\",\n    \"protocol\":\"http\",\n    \"data\": a\n}\nreturn msg;",
    "outputs": 1,
    "noerr": 0,
    "initialize": "",
    "finalize": "",
    "x": 420,
    "y": 200,
    "wires": [["66c43e6.4f5f7c"]]
  }
]
```

在 Azure IoT Hub node 中設定 IoT Hub 的 hostname，並指定想要使用的 protocol

![](https://imgur.com/At881eD.png)

並在 msg.payload 中以 JSON 的形式指定下面訊息：

```js
msg.payload = {
  deviceId: "device1",
  key: "pOX7pNKnt2aJoTy5JX4BYXidsezO+fr1sEz0TQQt1YM=",
  protocol: "http",
  data: data,
};
```

按下 Deploy 後便可以看到資料已經成功送出

![](https://imgur.com/gbgRCqm.png)

## 準備 Cloud Node-red 的 workflow

為了將收到的資料呈現在 dashboard，在 manage palette 中安裝了 node-red-dashboard。

![](https://imgur.com/QxkXcvr.png)

準備　 Azure IoT Hub Receiver 接收 IoT Hub 中的資料，並接上 chart node，以 line chart 的方式將收到的資料呈現在 dashboard 上。

![](https://imgur.com/wj3pOii.png)

```json
[
  {
    "id": "abefc0d0.8ba21",
    "type": "azureiothubreceiver",
    "z": "3664d073.9332c",
    "name": "Azure IoT Hub Receiver",
    "x": 360,
    "y": 180,
    "wires": [["ac07fe9f.05a0c"]]
  },
  {
    "id": "935ce90f.e63358",
    "type": "debug",
    "z": "3664d073.9332c",
    "name": "Log",
    "active": true,
    "tosidebar": true,
    "console": false,
    "tostatus": false,
    "complete": "payload",
    "targetType": "msg",
    "statusVal": "",
    "statusType": "auto",
    "x": 810,
    "y": 180,
    "wires": []
  },
  {
    "id": "ac07fe9f.05a0c",
    "type": "ui_chart",
    "z": "3664d073.9332c",
    "name": "",
    "group": "9f102f5f.29f03",
    "order": 0,
    "width": "0",
    "height": "0",
    "label": "chart2",
    "chartType": "line",
    "legend": "false",
    "xformat": "HH:mm:ss",
    "interpolate": "linear",
    "nodata": "",
    "dot": false,
    "ymin": "0",
    "ymax": "100",
    "removeOlder": 1,
    "removeOlderPoints": "",
    "removeOlderUnit": "3600",
    "cutout": 0,
    "useOneColor": false,
    "useUTC": false,
    "colors": [
      "#1f77b4",
      "#aec7e8",
      "#ff7f0e",
      "#2ca02c",
      "#98df8a",
      "#d62728",
      "#ff9896",
      "#9467bd",
      "#c5b0d5"
    ],
    "useOldStyle": false,
    "outputs": 1,
    "x": 610,
    "y": 180,
    "wires": [["935ce90f.e63358"]]
  },
  {
    "id": "9f102f5f.29f03",
    "type": "ui_group",
    "z": "",
    "name": "Default",
    "tab": "a79849a0.efb138",
    "order": 1,
    "disp": true,
    "width": "15",
    "collapse": false
  },
  {
    "id": "a79849a0.efb138",
    "type": "ui_tab",
    "z": "",
    "name": "Home",
    "icon": "dashboard",
    "disabled": false,
    "hidden": false
  }
]
```

Deploy 之後可以看到 dashboard 成功呈現。

![](https://imgur.com/vpVFzyT.png)
