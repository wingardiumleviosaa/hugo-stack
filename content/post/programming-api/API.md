---
title: "[API] 簡介及實作 & 資料交換格式 - XML, JSON, SOAP"
categories: ["Programming"]
date: 2020-07-02 21:10:00
slug: "api-data-transform"
---

## API

Application Programming Interface 應用程式介面，其中『介面』就是溝通的管道，比如說 USB 隨身碟的介面就是 USB．所有電腦以及 USB 隨身硬碟的廠商只要按照 USB 的規則製造就能使用此介面去溝通。

<!--more-->

## 提供 API & 使用 API

比如電腦使用者說想知道網路狀況，作業系統底層變提供了 API 讓我們使用。又比如說想要更改某個網站的會員資料，那這個網站就提供了 API 讓我們能在網站上修改資料庫的會員資料。或是要在網頁上加入 google map 的功能，就需使用 google map 提供的 API。

{{< quote>}}
所以 API 可以想成是某種功能，用以溝通兩個不同的東西用的。
{{< /quote >}}

## WebAPI

指 HTTP API，透過 HTTP 協定的 API。通常透過 HTTP method 去呼叫 API 進而交換資料，基本上就是丟 request，然後拿到 response 的資料。

## SDK

Software Development Kit 軟體開發套件，是用來開發特定應用程式的工具組，通常是廠商針對某一平臺、系統、或硬體所釋出，用以開發應用程式的工具組，在這個工具包裡面，可能包含了各式各樣的開發工具，模擬器或 API 等。

## api 串接實作

將使用 [reqres](https://reqres.in/) 網站當作伺服器，下 API 去取得該網站的資料。另外使用 request node module 實作。

1. 發 request 到網址，並印出 body

![](https://imgur.com/R4FKQdA.png)

```js
const request = require("request");
request("https://reqres.in/api/users?page=2", function (error, response, body) {
  if (error) {
    return console.log("request failed", err);
  }
  console.log(body);
});
```

印出所有 user list

![](https://imgur.com/aRf6k3X.png)

2. 發 request 到網址，並印出單一使用者資料

![](https://imgur.com/i9dXR3c.png)

```js
const request = require("request");
request("https://reqres.in/api/users/2", function (error, response, body) {
  if (error) {
    return console.log("request failed", err);
  }
  console.log(body);
});
```

![](https://imgur.com/mJQUD1u.png)

3. 把取得得特定使用者變成是下 node index.js 時的參數
   將使用另一個 node module - process

```js
const request = require("request");
const process = require("process");
console.log(process.argv);
request(
  "https://reqres.in/api/users/" + process.argv[2], //第三個參數
  function (error, response, body) {
    if (error) {
      return console.log("request failed", err);
    }
    console.log(body);
  }
);
```

![](https://imgur.com/H5CQh3z.png)

4. post 一筆新使用者資料到網站上

![](https://imgur.com/j4UE1Zp.png)

```js
const request = require("request");
request.post(
  {
    url: "https://reqres.in/api/users/",
    form: {
      name: "ula",
      job: "student",
    },
  },
  function (error, response, body) {
    if (error) {
      return console.log("request failed", err);
    }
    console.log(response.statusCode); //印出 http 狀態碼
  }
);
```

![](https://imgur.com/XQcAPCK.png)

5. 使用 patch 修改資料

```js
const request = require("request");
request.post(
  {
    url: "https://reqres.in/api/users/2",
    form: {
      name: "ula",
      job: "student",
    },
  },
  function (error, response, body) {
    if (error) {
      return console.log("request failed", err);
    }
    console.log(response.statusCode); //印出 http 狀態碼
  }
);
```

6. 使用 delete 刪除資料

```js
const request = require("request");
request.delete(
  "https://lidemy-book-store.herokuapp.com/books/21",
  function (error, response, body) {
    if (error) {
      return console.log("request failed", err);
    }
    console.log(response.statusCode); //印出 http 狀態碼
    console.log(body);
  }
);
```

---

## CURD

CURD 是創建（Create）、更新（Update）、讀取（Read）和刪除（Delete）操作的縮寫。在電腦程式語言中是一連串常見的動作行為，通常是為了針對某個特定資源所作出的舉動（例如：建立資料、讀取資料等）。這四種行為最常使用在 SQL 資料庫操作或網站的 API 串接時。

## RESTful API

REST (Representational State Transfer)具象狀態傳輸，而 Restful 形容詞， Restful API 是一種風格而非協定，形容以此規範設計的 API。

RESTful 風格的網址設計強調從 URL 就能看出要對什麼資料(資源名稱)、進行什麼操作(HTTP Method)。

- 瀏覽全部資料：GET + 資源名稱
- 瀏覽特定資料：GET + 資源名稱 + id
- 新增一筆資料：POST + 資源名稱
- 修改特定資料：PUT + 資源名稱 + id
- 刪除特定資料：DELETE + 資源名稱 + id

例如前面的 reqres 範例，get https://reqres.in/api/users/2，取得第二筆 users 資料。

<table><tr><td bgcolor=AliceBlue>
<b>小補充</b></p>
API 的網址通常會定義如下，會為 api 加上版本資訊，以便未來改版時不同版本的 client 可以同時使用。

```
https://ulagraphy.com/api/v1
```

</td></tr></table>

---

## 資料格式

### XML

Extensible Markup Language，跟 html 很像，是一種標記式語言。與 html 不同的是，XML 可以允許使用者自行定義所需的標籤(tags)，主要的功能是用來「資料傳遞」用。

![](https://imgur.com/h2QBAaD.png)

### JSON

Javascript Object Notation，是一種資料格式。相較於 XML，近年來 JSON 較廣為使用，易讀且檔案大小相對較小。
前面的範例中 response 回傳的資料雖然是字串，但可以被轉成 json。

```js
const request = require("request");
const process = require("process");
request(
  "https://reqres.in/api/users/" + process.argv[2], //第三個參數
  function (error, response, body) {
    const json = JSON.parse(body);
    console.log(json);
  }
);
```

![](https://imgur.com/e1lJJHm.png)

轉成 json 格式後就可以存取裡面的屬性值。

```
console.log(json.data.first_name)
```

![](https://imgur.com/fvitz2E.png)

<table><tr><td bgcolor=AliceBlue>
<b>補充</b></p>
1. 字串轉 JSON 使用  `JSON.parse(str)` </br>
2. Javascript 物件轉 JSON 格式的字串可使用  `JSON.stringify(obj)`  </br>
3. 任何一種資料格式在任何程式語言都可以使用，所以雖然 JSON 全名有 Javascript，但他也支援其他如 C, Pyhon 等語言
</td></tr></table>

### SOAP

除了前面提到的 HTTP METHOD 可以下 API 來跟網頁溝通外，還有其他種方式，SOAP (Simple Object Access Protocol) 就是其一，SOAP 的交換(request & response)都是透過 XML。Node.js 有提供 node-soap 的 module，方便產生 SOAP 的資料格式，但此協定已很少被使用了！

---

## reference

- [restful-api](https://noob.tw/restful-api/)
- [Wiki-CURD](https://zh.wikipedia.org/wiki/%E5%A2%9E%E5%88%AA%E6%9F%A5%E6%94%B9)
- 記此篇為觀看 Lidemy NET101 的筆記，部分圖片以及內容取自上課影片
