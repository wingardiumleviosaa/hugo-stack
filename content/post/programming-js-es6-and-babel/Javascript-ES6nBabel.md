---
title: "[Javascript] ES6 新語法 & babel 轉譯器簡介"
tags:
  - Javascript
  - Node.js
categories:
  - Programming
date: 2020-06-27 13:39:00
slug: js-es6-and-babel
---

## ECMAScript

[ECMAScript](https://zh.wikipedia.org/wiki/ECMAScript)是一個開發標準(規範)，而 Javascript 就是根據此標準來實作。
ES6 指的是 ECMAScript 第六版，於 2015 年發布，所以又稱 ES2015。ES6 包含了許多新的語法，將於下面用 Javascript 介紹。

<!--more-->

## 新的變數宣告方式

1. const：用於宣告不會變的常數。
   但如果今天宣告的常數是物件的話，因為物件的底層存的是位址，所以即使宣告的方式為 const 但物件內的屬性值是可以改變的。
2. let：使用 let 宣告的變數的生存範圍 (scope) 僅限於變數所在的 block 中。

```js
function test() {
  if (10 > 5) {
    var a = 10;
    let b = 20;
    console.log(b);
  }
  consle.log(a);
}
```

b 的生存範圍僅限於 if 區塊中；而 a 的生存範圍則是整個 function。

{{< notice info >}}
作用域愈小愈好，才不會干擾到其他人，所以比較建議使用 let。
{{< /notice >}}

## 更方便的串接字串 - Template Literals

字串串接使用 `` ` ` `` 取代原先使用的單引號 `' '` 或是雙引號 `" "`。

```js
//以前
var name = "jack";
var str = "hello" + name + ", it is" + new Date() + "now!";
var str1 = "hello\nworld\nhello\nJavascript";

//ES6
var newStr = `hello ${name.toUpperCase()}, it is ${new Date()} now!`;
var newStr1 = `hello
world,
hello
Javascript!`;
```

以前在使用單雙引號串接字串的時候，如果遇到換行或是要跟字串相接的狀況時需要使用加號串，很麻煩！  
現在 ES6 提供反引號來串接，當遇到變數的時候只需要在反引號內使用 `${變數}` 或 `${javascript code}` 包起來，就可以不必再打一大堆加號跟引號了。

## 解構 Destructuring

陣列或物件的元素可以使用解構指定給變數

### 解構陣列

```js
const arr = [1, 2, 3, 4];
var [first, second, third, fourth] = arr; // 1, 2, 3, 4
var [one, two] = arr; // 1, 2
```

### 解構物件

```js
const obj = {
  name: "jack",
  age: 20,
  address: "taiwan",
  family: {
    father: "david",
  },
};
var { name, age } = obj;
var { family } = obj; //{father : 'david'}
var { father } = family; //david，再次解構 family 這個物件
```

- 陣列解構使用 `[]`
- 物件解構使用 `{}`，且變數需與物件屬性**相同**名稱
- 解構可以部份解構或雙層解構，會自動對應
- 部分解構如第三行，只寫兩個變數，就會自動對應前兩個元素
- 雙層解構如最後兩行，可以直接變成一行 `var {family : {father}} = obj`

補充另一個很酷的用法！

在以前函數傳物件的時候要使用裏面的變數:

```js
function test(obj) {
  console.log(obj.a);
}
test({
  a: 1,
  b: 2,
});
```

現在有了解構的話就可以這樣寫：

```js
function test({ a, b }) {
  console.log(a);
}
test({
  a: 1,
  b: 2,
});
```

## 展開運算子 Spread Operator

### 用 `...` 展開陣列。

```js
var arr = [1, 2, 3];
var arr2 = [4, 5, 6, arr]; //[4, 5, 6, [1, 2, 3]]
var arr3 = [4, 5, 6, ...arr]; //[4, 5, 6, 1, 2, 3]

function add(a, b, c) {
  return a + b + c;
}
console.log(add(...arr));
```

### 用 `...` 展開物件。

```js
var obj1 = {
  a: 1,
  b: 2,
};
var obj2 = {
  ...obj1,
  c: 3,
};
var obj3 = {
  ...obj1,
  b: 3,
};
var obj3 = {
  b: 2,
  ...obj1, //在後面的變數會蓋掉前面的
};
console.log(obj2); //{a : 1, b : 2, c : 3}
console.log(obj3); //{a : 1, b : 3}
console.log(obj3); //{b : 2, a : 1}
```

### 使用 `...` 來複製**值**

陣列放入展開的陣列會是一個擁有不同位址的新陣列，所以要單純複製值的話，要使用展開運算。

```js
var arr = [1, 2, 3];
var arr1 = arr; //傳址
var arr2 = [...arr]; //傳值
console.log(arr === arr1); //輸出 true
console.log(arr === arr2); //輸出 false
```

下面很酷：

```js
var nest = [4];
arr1 = [1, 2, 3, nest];
arr2 = [...arr1];
console.log(arr1[3] === arr2[3]); //輸出 true
```

arr1 中的 nest 是用傳址放進來的，所以儘管 arr2 使用展開的 arr1，arr2 中的 `[4]` 也還會是同一個位址。

### 其餘參數 Rest Parameter

可以理解成『反向』展開，通常和解構一起使用，必須放在最後面。

#### 應用在陣列及物件上

```js
var arr = [1, 2, 3, 4];
var [first, ...rest] = arr;
console.log(first, rest); //1 [2, 3, 4]

var obj = {
  a: 1,
  b: 2,
  c: 3,
};
var { a, ...obj2 } = obj;
console.log(obj2); //{b:2, c:3}
```

#### 應用在函式的參數上

```js=
function add(...args){
    console.log(args)
    return args[0]+args[1]
}
console.log(add(1,2))
```

## 預設值 Default Parameters

在 function 的參數加入預設值，當在呼叫時沒有給相對應的參數時，就會用預設值當值傳入。

```js
function repeat(str = "ya", times = 3) {
  return str.repeat(times);
}
console.log(repeat("abc"));
```

## 箭頭函式 arrow function

可以簡化函式的寫法

```js
const test = function (n) {
  return n;
};
```

簡化成

```js
const test = (n) => {
  return n;
};
```

如果只有一個參數的話 () 可以省略掉；
如果在 return 就能做事並回傳的話 {} 可以省略掉。

```js
var arr = [1, 2, 3, 4, 5];
console.log(
  arr
    .filter(function (value) {
      return value > 3;
    })
    .map(function (value) {
      return value * 2;
    })
); //輸出 [8, 10]
```

可以省略成

```js
var arr = [1, 2, 3, 4, 5];
console.log(arr.filter((value) => value > 3).map((value) => value * 2));
```

## export & import 的新寫法

### 個別 export

```js
//app.js
export const PI = 3.14;
export function add(a, b) {
  return a + b;
}
```

### 統一 export

```js
//app.js
const PI = 3.14;
function add(a, b) {
  return a + b;
}
export {
  add as addFunction, //使用 as 可以改名
  PI,
};
```

### 指名 import

```js
//index.js
import { PI, addFunction as a } from "./app.js";
//import 也可以使用 as 改名
console.log(a(3, 5), PI);
```

### 用 \* 全部 import

```js
//index.js
import * as app from "./app.js";
console.log(app.addFunction(3, 5), app.PI);
```

### export default & import default

可以不用加大括號

```js
//app.js
export default function add(a, b) {
  return a + b;
}
export var PI = 3.14;
```

</br>

```js
//index.js
import add, { PI } from "./app";
```

p.s. Node.js 還沒支援這種新語法，所以要使用接下來介紹的轉譯器。

## Babel

Babel 是 JavaScript 轉譯器，可將 ES6 以上的程式碼轉為 ES5 程式碼，以支援還沒支持新語法的瀏覽器或環境。

{{< notice warning >}}
babel-node 執行的時候會佔大量的記憶體空間，官方不建議在 production 環境使用。
{{< /notice >}}

1. 安裝 babel-node
   `$ install --save-dev @babel/core @babel/node`

2. 安裝 presets 並配置 .babelrc 文件
   因為 babel-node 對 import 語法默認是關閉的，因此需要安裝指定的 preset 並配置 .babelrc 文件來開啟語法支援。
   `$ npm i @babel/preset-env --save-dev`
   新增 .babelrc 配置文件 `$ touch .babelrc`，並在文件中貼上：
   `$ vi .babelrc`

```
{
  "presets": [ "@babel/preset-env" ]
}
```

3. 執行 babel-node
   `npx babel-node index.js`

## Source

[All] 此篇為觀看 Lidemy JS102 的筆記，圖片來源以及部分內容取自上課影片
