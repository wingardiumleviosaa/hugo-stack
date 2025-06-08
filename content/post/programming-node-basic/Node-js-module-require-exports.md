---
title: "[Node.js] module、require & exports"
tags:
  - Node.js
categories:
  - Programming
date: 2020-06-25 22:03:00
slug: node-module-require-and-exports
---

## Module/Package/Liberary

當程式裡面有很多個功能，可以把它拆開成一個一個的模組，之後才方便維護，因為彼此的相依性被獨立開來了，而且同一個功能不用一直重寫，只要把 module 匯入就好。

<!--more-->

另外還可以在程式中使用別人寫好的 module，以減少開發時間。

![](https://imgur.com/TSHVodQ.png)

## 在程式裡使用 node module - require

require 中直接寫要引入的 module 名稱，然後取一個變數給它。

```js
var os = require("os");
var http = require("http");
```

如果是自己寫的非官方 module，在 require 時沒有加路徑的話會先從本身檔案的同路徑下找 (./)，沒有找到的話會去 node-module 資料夾裡找。

## 把 module 借給別人用 - exports

```js
// app.js
function double(n) {
  return n * 2;
}

module.exports = double;
```

在其他程式呼叫時，就是 module.exports 輸出的東西，意即 module.exports 丟什麼，require 時的變數就會是什麼東西。

```js
// index.js
var double = require("./app.js");
console.log(double(3)); //輸出 6
```

其中：

1. require 的檔案要指名路徑
2. 可省略副檔名(一般都是不打居多)

但在大部分的場合下我們不會只輸出一個東西(function , variable...)，所以用以下兩種方式

### module.exports={物件}

```js
module.exports={
    double: double
    triple: function(n){
        return n*3
    }
}
```

在 require 端就可以：

```js
var app = require("./app");
console.log(app.double(2));
console.log(app.triple(3));
```

### exports.<輸出的東西>

```js
exports.double = double;
exports.triple = function (n) {
  return n * 3;
};
```

在 require 端就可以：

```js
var app = require("./app");
console.log(app.double(2));
console.log(app.triple(3));
```

</br>

{{< notice info >}}
使用 export.<sth> 輸出的東西，都會是物件！ 而使用 module.export 輸出可以是數值、陣列、函式、或物件。
{{< /notice >}}

## Source

[All] 此篇為觀看 Lidemy JS102 的筆記，圖片來源取自上課影片
