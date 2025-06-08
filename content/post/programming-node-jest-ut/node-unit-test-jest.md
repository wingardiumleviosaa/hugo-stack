---
title: "[Node.js] 單元測試 & Jest 簡介"
tags:
  - Node.js
categories:
  - Programming
date: 2020-06-25 22:11:00
slug: node-unit-test-jest
---

## 單元測試

單元測試（Unit Testing），是針對專案中每一個單一功能做測試。一般專案裡的最小單位是一個 function，而通常我們在寫完程式後都是用 console.log 來確認呼叫的結果：

<!--more-->

```js
//index.js
function repeat(str, n) {
  var result = "";
  for (var i = 0; i < n; i++) {
    result += str;
  }
  return result;
}

console.log(repeat("a", 5) === "aaaaa"); //輸出 true
```

## Jest

Jest 是一套現成測試的框架

1. `$ npm install jest --save-dev` 下載
2. 將要測試的程式 export 成 module

```
//index.js
//上半部不變，刪掉原先的測試用的 console.log 這行，加上：
exports.repeat = repeat
```

3. 新增 jest 測試檔案， 使用`<NAME>.test.js` 當作測試檔檔名

```js
// index.test.js
var repeat = require('./index.js')
test('repeat('a',5)應該要等於'aaaaa'', () => {
    expect(repeat('a', 5).toBe('aaaaa'));
});
```

其中 () =>{} 是 function(){} 的縮寫 4. 在 package.json 的 script 中加上 jest

```json
"script":{
    "test" : "jest"
}
```

5. 下 `$ npm run test` 就會自動去找附檔名為 .test.js 的檔案並跑測試

6. 或是只是想測試單一的檔案，則在 script 中指明檔案即可

```json
"script":{
    "test" : "jest repeat.test.js"
}
```

## 加上 describe 歸納相同性質的測試

![](https://imgur.com/Ea7Y3S3.png)

![](https://imgur.com/fVkl5FZ.png)

## 補充

如果直接在 terminal 下 jest index.test.js 指令的話，會出現 command not found 的錯誤訊息。因為 jest 只安裝在專案底下，而下在 terminal 是去你的系統找，所以會有錯誤。
使用 package.json 的 script 來下指令外還可以使用 `npx` 來達成直接在 terminal 下 jest 的功能，`$ npx jest repeat.test.js`。

## TDD, 測試驅動開發

Test-driven Development，一種開發方法，會先寫出測試程式，在依預期的測試結果去開發程式。

## Source

[All] 此篇為觀看 Lidemy JS102 的筆記，圖片來源以及部分內容取自上課影片
