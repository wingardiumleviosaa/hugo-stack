---
title: "[HTTP] content-type"
categories: ["SRE"]
tags: ["Network"]
date: 2020-07-21 21:20:00
slug: "http-contenttype"
---

## 定義

網際網路媒體類型（Internet media type，也稱為 MIME 類型（MIME type）或內容類型（content type））是給網際網路上傳輸的內容賦予的分類類型。HTTP 透過 `content-type` 的 header 來表示 request 或 response message 的 body 是用何種方式編碼，伺服器會根據編碼類型使用特定的解析方式，獲取數據流中的數據。

<!--more-->

其格式為

```
類型名/子類型名; 可選参数
```

比如

```
text/html; charset = UTF-8
```

## 常見類型

### application/x-www-form-urlencoded

- 最常見的 POST 提交數據方式，數據以鍵值 key1=val1&key2=val2 的方式編碼（urlencoded），編碼主要用來轉換易混淆的字串如 `&`、`=`。
- 瀏覽器原生表單 `<form>` 默認的提交方式（沒有特別設 enctype 屬性的話）。
- jquery 默認 post 請求提交的方式。

![](https://imgur.com/g5ZON7t.png)

### multipart/form-data

不會對字元編碼，在使用包含檔案上傳控制元件的表單時，則必須使用該值`<form enctype="multipart/form-data">`。

會生成一個複雜的 boundary 字串來分割不同的字段。

```
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"
title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png
PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

Body 中按照字段個數可分為多個相同結構的部分，每部分都是以 `--boundary` 開始，接著內容描述信息，換行，最後是字段具體內容（文本或二進制）。Body 最後以 `--boundary--` 標示結束。

### text/plain

數據以純文本形式(text/json/xml/html)進行編碼，其中不含任何控制項或格式字符。

{{< notice warning >}}
以上三種 content type，是 HTML5 規範中 form 的 enctype 的可能值。
{{< /notice >}}

### application/json

用 json 來傳遞參數資料，可以方便的提交複雜的結構化數據，特別適合 RESTful 的接口。

{{< notice warning >}}
application/json 已經被 W3C 遺棄，建議不要在`<form enctype="...">`中使用，即使用了如果瀏覽器不支持，也會替換成 application/x-www-form-urlencoded。
同理，其余的 MIME 類型，也不支持，均會替換成默認編碼 application/x-www-form-urlencoded。
{{< /notice >}}

## Reference

- [1]https://imququ.com/post/four-ways-to-post-data-in-http.html
- [2] https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91%E5%AA%92%E4%BD%93%E7%B1%BB%E5%9E%8B
- [3] https://www.itread01.com/content/1516643918.html
