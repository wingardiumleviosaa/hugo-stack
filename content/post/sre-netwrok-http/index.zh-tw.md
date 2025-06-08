---
title: "[TCP/IP] 應用層 - HTTP"
categories: ["SRE"]
tags: ["Network"]
date: 2020-07-01T10:24:00+08:00
slug: "tcpip-http"
---

## HTTP

HyperText Transfer Protocol 超文本傳輸協定，是網頁的基礎通訊協定，基於 client-server 架構來 request-reponse，用來在 client 以及 server 間傳遞超媒體文件(如 HTML) 。

<!--more-->

### 運作模型

1. client 傳 **http request** 到 server
2. server 回傳 **http response** (html)

![](https://imgur.com/D8HQAS4.png)
[1]

- HTTP Client 實現程式有瀏覽器、命令列工具 ... 等
- HTTP Server 實現程式有 httpd、nginx、apache ... 等
- https 的 s 代表 secure，是一個比 http 更安全的連線方式

## Http Method

| METHOD | 安全性 | 動作     | 語意                                                                       |
| ------ | ------ | -------- | -------------------------------------------------------------------------- |
| GET    | O      | 讀取     | 請求所需要的資源。                                                         |
| POST   | O      | 新增     | 在請求中攜帶 request body，並執行特定資源的處理。                          |
| HEAD   | X      | 讀取     | server 僅回應狀態以及 header。                                             |
| PUT    | x      | 完整更新 | 請求更新一筆資源的所有內容，必須是存在的資源，資源傳遞必須完整，否則為空。 |
| PATCH  | X      | 部分更新 | 請求更新一筆資源的部分內容，必須是存在的資源。                             |
| DELETE | x      | 刪除     | 請求移除資源。                                                             |

[2]

## Http 狀態碼

狀態碼通常都以開頭的數字做區分，例如 2 開頭的狀態碼都是代表成功。

- 1xx informationa - 參考資訊
- 2xx Successful - 成功
- 3xx Redirection - 重新導向
- 4xx Client Error - 用戶端錯誤
- 5xx Server Error - 伺服器錯誤
  詳細代碼可參考 [wiki](https://zh.wikipedia.org/wiki/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)。

## 自己的 request 自己發 - 實作 http client

前面有提到實現 http client 有瀏覽器還有其他諸如 CLI、爬蟲 ... 的程式。這邊我們來實作一個使用 [request](https://www.npmjs.com/package/request) node module 的 http client。

1.  `$ npm install request`
2.  在 index.js 檔案中貼上

![](https://imgur.com/PTiqaMj.png)

3.  `$ node index.js`，就可以收到 server 的回覆

![](https://imgur.com/6nI08to.png)

## 架一個簡易 http server

使用 Node.js 的內建模組 `http`，

```js
var http = require("http");
var server = http.createServer(handelRequest);
function handelRequest(req, res) {
  if (req.url === "/") {
    //根據 request 的資源
    res.write("welcome!"); //給 相對應的 response
    res.end();
    return;
  }
  if (req.url === "/hello") {
    res.write("hello");
    res.end();
    return;
  }
  if (req.url === "/redirect") {
    res.writeHead(302, {
      //如果 url 是 /redirect 的話則轉址到 /hello
      Location: "/hello",
    });
    res.end();
    return;
  }
  res.writeHead(404);
  res.end();
}
server.listen(5000); //port
```

`$ node index.js` 執行後，開啟瀏覽器輸入 `127.0.0.1:5000`，便可以看到 response 的結果。

![](https://imgur.com/Ps8Jvz5.png)

第 14 行，如果網址結尾是 /redirect 的話，轉到其他位址。

![](https://imgur.com/AAFk2Yw.png)

第 21 行，代表如果帶錯誤的 url 的話，回復 404 status code。

![](https://imgur.com/GlF8sd2.png)

## Reference

[1]https://techthatmatter.com/best-fastest-bsnl-dns-servers/  
[2]https://hackmd.io/@monkenWu/Sk9Q5VoV4/https%3A%2F%2Fhackmd.io%2F%40gen6UjQISdy0QDN62cYPYQ%2FH1yxwXyNN?type=book
