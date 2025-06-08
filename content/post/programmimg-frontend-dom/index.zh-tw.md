---
title: "[Frontend] DOM"
categories:
  - Programming
tags: ["HTML"]
date: 2020-10-06 15:11:00
slug: "html-dom"
---

DOM (Document Object Model) 定義了標準 API，使 JavaScript 可以控制瀏覽器的行為與網頁的內容。

<!--more-->

DOM 將一份 HTML 文件看作是一個樹狀結構的物件，裡面的標籤皆為節點 (node)，可以改變其結構、樣式 (CSS) 或內容等：

![](https://imgur.com/Mkx74z0.png)

## 節點類型

DOM 物件模型把 HTML 的元素 (element) 當作是 JavaScript 物件 (object) 來操作，常見的節點類型---

- document：DOM tree 的根節點，所以當要存取 HTML 時，都從 document 物件開始。
- element
- event
- window

參考: https://pydoing.blogspot.com/2011/08/javascript-htmldom-overview.html

## DOM API 的使用範例

```js
// 取得頁面上所有的 <p> 元素
var paragraphs = document.getElementsByTagName("p");

// 將所有的 <p> 元素的文字顏色都改成綠色
for (var i = 0; i < paragraphs.length; ++i) {
  paragraphs[i].style.color = "green";
}
```

## Reference

- https://zh.wikipedia.org/wiki/%E6%96%87%E6%A1%A3%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B
- https://www.fooish.com/javascript/dom/
