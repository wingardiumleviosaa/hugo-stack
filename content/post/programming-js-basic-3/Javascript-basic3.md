---
title: "[Javascript] 函式、內建函式"
tags:
  - Javascript
categories:
  - Programming
date: 2020-06-17 22:35:00
slug: js-basic-3
---

## 函式 functiton

基本函式結構

```js
function 函式名稱(參數) {
  return 回傳值;
}
```

- 傳入參數可以一個或多個，可以是值、變數、陣列、物件或是函式
- 回傳值可以零個或一個，可以是一個值、變數、陣列、物件或是函式
- 呼叫函式直接打 `函式名稱()` ；如果有帶參數，將引數置於括號中 `函式名稱(引數)`

<!--more-->

## 參數 Parameter v.s. 引數 Argument

```js
funciton add(a,b){
    console.log(arguments)  //{'0':2, '1':5}
    console.log(arguments[0]) //2
    return a+b
}
add(2,5)
```

其中 a, b 為 function add 的`參數`；2, 5 為`引數`。
Javascript 提供了一個引述的變數 `arguments`，可以在 function 內使用。

## 小補充

arguments 是一個`物件`，但你可能會疑惑為什麼可以用類似陣列的方式 `arguments[0]` 存取，因為上一篇[文章](https://ulahsieh.netlify.app/p/js-basic-1/)物件有提到，存取物件屬性值可以用`物件.屬性名`或是`物件['屬性名']`，而 argument['0'] 中的字串`'0'`，可以直接用數字 `0` 表示，所以就會看起來想存取陣列的表示方式了。

### 不同的函式宣告方式

1. 直接宣告一個具名的 function

```js
function hello() {
  console.log("hello");
}
```

2. 宣告變數等於一個匿名 function

```js
var hello = function () {
  console.log("hello");
};
```

3. function 當參數傳入另一 function

```js
function print(anything) {
  anything();
}
function hello() {
  console.log("hello");
}
print(hello);
```

</br>

```js
function trans(x, passInFunc) {
  return passInFunc(x);
}
function double(x) {
  return x * 2;
}
console.log(trans(10, double));
```

</br>

```js
function trans(x, passInFunc) {
  return passInFunc(x);
}

console.log(
  trans(10, function (x) {
    return x * 2;
  })
); //傳入匿名函式當參數
```

## function 的傳值與傳址

```js
function swap(a, b) {
  var temp = a;
  a = b;
  b = temp;
  console.log("a, b:", a, b);
}
var num1 = 10;
var num2 = 20;
console.log(num1, num2);
swap(num1, num2);
console.log(num1, num2);
```

執行結果如下

```
10 20
a, b: 20 10
10 20
```

在呼叫 swap(num1, num2) 時，是把 num1, num2 的**值**複製一份到 swap(a, b)，而非位址，所以 num1 之於 a 是兩個位址不一樣的變數。

<font color=darkred>但是如果傳入的是物件或是陣列的話，就是傳址。</font>

![](https://imgur.com/j1kZHAi.png)

```js
function addValue(obj) {
  obj.num++;
  obj.num2 = 20;
}
var a = {
  num: 1,
};
addValue(a);
console.log(a);
```

執行結果如下

```
{ num : 2, num2 : 20}
```

但是如果不是用`物件.屬性`去改值的話，而是去賦予一個新的物件，那底層的指標就會改位址了。

![](https://imgur.com/0G1ZvMf.png)

```js
function addValue(obj) {
  obj = {
    num: 100,
  };
}
var a = {
  num: 1,
};
addValue(a);
console.log(a);
```

執行結果如下

```
{ num : 1 }
```

詳細參考[文章](https://blog.techbridge.cc/2018/06/23/javascript-call-by-value-or-reference/)

## return 與不 return

要不要 return 取決於需不需要知道結果，呼叫的方式就會不同。
有帶 return 的 function 呼叫時會宣告一個變數放回傳值；

```js
function addNum(a) {
  return a + 1;
}
var num = printNum(15);
console.log(num); //16
```

warning
function 一旦 return 後．return 後的程式碼就都不會執行了。

例如下面的範例只會印出第七行，不會印出第四行，因為遇到前一行的 return 就返回了。

```js
function addNum(a) {
  a++;
  return a;
  console.log(a);
}
var num = printNum(15);
console.log(num); //16
```

如果是沒有 return 的函式，通常都直接呼叫即可。因為就算賦予給一變數，變數的內容也會是 undefined（javascript 預設）。

```js
function printNum(a) {
  console.log(a);
}
printNum(15); //15
var num = printNum(15);
console.log(num); //undefined
```

## 數字相關的內鍵函式

### 字串轉數字

- Number(n)
  ```js
  var a = 10;
  var b = "20";
  console.log(a + Number(b)); //輸出 30
  ```
- parseInt(n, 進位數)
  ```js
  var a = 10;
  var b = "20.35";
  console.log(a + parseInt(b, 10)); //輸出 30
  ```
- parseFloat(n, 進位數)
  ```js
  var a = 10;
  var b = "20.35";
  console.log(a + parseFloat(b, 10)); //輸出 30.35
  ```
- parseFloat(n, 進位數).toFixed(四捨五入截到小數點後第幾位)
  ```js
  var a = 10;
  var b = "20.3523546";
  console.log(a + parseFloat(b, 10)); //輸出 30.35
  ```

### toString

可以使用 ‵.toString()‵ 函式將數字轉字串，或是加上 ‵''‵ 空字串。

```js
var a = 10;
var b = a.toString();
var c = a + "";
console.log(typeof a); //輸出 number
console.log(typeof b); //輸出 string
console.log(typeof c); //輸出 string
```

可以使用 toString 將數字轉成任意進位

```
console.log((30).toSting(2)) //輸出 11110
```

### 補充

- Number.MAX_VALUE

![](https://imgur.com/Sojie3F.png)

代表 Javascript 可以存的最大數字。

### Math

- Math.PI  
  圓周率，通常使用 const 宣告常數來賦值 `const pi = Math.PI`
- Math.ceil(n)  
  無條件進位
- Math.floor(n)  
  無條件捨去
- Math.round(n)  
  四捨五入
- Math.sqrt(n)  
  對 n 開根號
- Math.pow(n, x 次方)  
  回傳 n 的 x 次方 `console.log(Math.pow(2,10))`
- Math.random()  
  產生一個 0 到 1 (大於等於 0、小於 1) 的隨機數
  如果要取一個 1 ~ 10 的隨機整數：` Math.floor(Math.random()*10+1)`

## 字串相關的內鍵函式

### 字串大小寫相關

- 'ABC'.toLowerCase() //abc
- 'abc'.toUpperCase() // ABC
- 'A'.charCodeAt(0) //65
- String.fromCharCode(65) //A

### indexOf

`字串.indexOf('目標')` 用以找字串中是否存在目標，存在的話回傳目標的第一個索引值；不存在的話回傳 `-1`。

```js
var str = "hello im ula";
console.log(str.indexOf("im")); //輸出 6
console.log(str.indexOf("world")); //輸出 -1
```

### replace

`字串.replace('原字串','新字串')` 用以將字串中第一個出現的指定原字串取代成新字串。

```js
var str = "hello world".replace("l", "!");
console.log(str); //輸出 he!lo world
```

如果要全部取代，需要使用正規表達式`/'字串'/g`，其中 g 是指 global：

```js
var str2 = "hello world".replace(/l/g, "!");
console.log(str); //輸出 he!!o wor!d
```

### split

`字串.split('分割基準')`，將字串用分割基準分成陣列。

```js
var str = "hello world";
console.log(str.split(" ")); //輸出 ['hello','world']
```

### trim

用來去掉字串前後兩端的空白。

```js
var str = "    hello world   ";
console.log(str.trim()); //輸出 'hello world'
```

### length

`字串.length` 回傳字串長度。

```js
var str = "hello";
for (var i = 0; i < str.length; i++) {
  console.log(str[i]);
}
```

p.s. 字串也可以像陣列一樣使用 str[index] 來存取該索引的值。

## 陣列相關的內鍵函式

### join

`陣列.join('連接字串')` 用來將陣列的元素以連接字串連起來變成一個字串回傳。

```js
var arr = [1,2,3]
console.log(arr.join('&')) // 輸出 "1&2&3
"
```

### map

`陣列.map(函式)`將陣列中的每個元素都調用到指定函式內，並回傳一個新的陣列。

```js
var arr = [1, 2, 3];
function double(x) {
  return x * 2;
}
console.log(arr.map(double)); //輸出[2, 4, 6]

console.log(
  arr.map(function (x) {
    return x - 1;
  })
); //輸出 [0, 1, 2]

console.log(arr.map(double).map(double)); //輸出 [4, 8, 12]
```

- 可以丟函式或直接寫匿名函式
- 可以把 map 串起來成多個操作

### filter

`陣列.filter(函式)`將陣列中的每個元素都調用到指定函式內，並回傳一個元素皆符合函式描述的新陣列。

```js
var arr = [1, 2, 3, -6, -3, 5];
console.log(
  arr.filter(function (x) {
    return x > 0; //輸出 [1, 2, 3, 5]
  })
);
```

### slice

`陣列.slice(開始索引,結束索引)`，回傳一個包含開始索引、不包含結束索引的新陣列。

```js
var arr = [1, 2, 3, 4, 5, 6];
console.log(arr.slice(3)); //輸出[4, 5, 6]
console.log(arr.slice(3, 5)); //輸出[4, 5]
```

### splice

`陣列.splice(開始索引,刪除數量,插入)`，可以插入元素和刪除元素，**陣列本身會改變**，並回傳被刪除的元素陣列，。

```js
var arr = ["apple", "banana", "candy", "donut"];
var arr2 = arr.splice(2, 1, "cake");
console.log(arr); //輸出 ['apple', 'banana', 'cake, 'donut']
console.log(arr2); //輸出 ['candy']
```

### sort

`陣列.sort()`，將陣列重新按照**字串大小**做排序，不管陣列本身是不是字串，且**陣列本身會改變**。

```js
const arr = ["D", "B", "F", "A"];
arr.sort();
console.log(arr);
// 輸出 ["A", "B", "D", "F"]

const arr2 = [1, 30, 4, 21, 100000];
arr2.sort();
console.log(arr2);
// 輸出 [1, 100000, 21, 30, 4]
```

如果陣列元素是數字，想以數字排大小的話，需要在括號內加上判斷函式。

```js
var nums = [4, 2, 5, 1, 3];
nums.sort(function (a, b) {
  return a - b; //由小排到大。如果 <= 0，則 a, b 位置不變；如果 > 0，則 b 要跟 a 調換
});
console.log(nums); //輸出 [1,2,3,4,5]
nums.sort(function (a, b) {
  return b - a; //由大排到小。如果 <= 0，則 a, b 位置不變；如果 > 0，則 b 要跟 a 調換
});
console.log(nums); //輸出 [5,4,3,2,1]
```

### reduce

`陣列.reduce(callback[accumulator, currentValue], initialValue)`方法的參數是一個回呼函式，會將陣列中的每個元素呼叫一次回呼函式，並將回呼函式的傳回值當作下一次呼叫回呼函式的參數傳入。

- accumulator：累積 callback 回傳值的累加器
- currentValue：當前元素
- initialValue：於第一次呼叫 callback 時要傳入的累加器初始值。預設為陣列的第一個元素

```js
/*-------before--------*/

var arr = [1, 2, 3];
var total = 0;

for (let i = 0; i < arr.length; i++) {
  total += arr[i];
}

/*-------after--------*/

var arr = [1, 2, 3];

var total = arr.reduce(function (total, x) {
  return total + x;
});

//或是 arr.reduce((total,x)=>total+x));
```

### forEach

`陣列.forEach(function callback(x){...}` 方法會將陣列內的每個元素傳入並執行給定的函式一次，不會回傳任何東西。

```js
/*-------before--------*/

var arr = [1, 2, 3];
var copy = [];

for (let i = 0; i < arr.length; i++) {
  copy.push(arr[i]);
}

/*-------after--------*/

var arr = [1, 2, 3];
var copy = [];

arr.forEach(function (x) {
  copy.push(x);
});
```

### Array().fill()

`Array(n).fill(0)` 產生一個 n 個長度的 array，每個元素都填零。
下面有一個看 [JS101] 印出很多星星的不同的超酷寫法：

```js
function star(n) {
  var result = "";
  for (var i = 0; i < n; i++) {
    result += "*";
  }
  return result;
}

function makeStars(layer) {
  return Array(layer)
    .fill(0)
    .map(function (x, index) {
      return star(index + 1);
    })
    .join("\n");
}

console.log(makeStars(5));
```

## Immutable Value 不可變

除了物件和陣列以外的 primitive type 皆是不可變的，不可變的意思是當宣告變數後，每一次的重新賦值都不是從原本的記憶體位址改，而是創造一個新的記憶體存新的值，舊的不可變。

![](https://imgur.com/8IrsAgr.png)

```
var a = 'hello'
a.toUpperCase()
console.log(a) //輸出 hello，因為第一行不可變
a = a.toUpperCase() //需要創造一個新的位置，重新賦值給 a
console.log(a)
```

而物件跟陣列**通常**都是可變的，例如 arr.push 會改動原本 arr 的陣列，但通常代表還是有例外，例如 arr.join。

因為實在有太多函式可以使用，但如果不確定到底是不是可變的，那建議在使用之前查一下[資料](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)。

## 遞迴

在 function 中一直重複呼叫自己，但前提是要有終止條件(return)。
JS101 費式數列

```
function fib(n){
	if(n===0) return 0
    if(n===1) return 1
    return fib(n-2) + fib(n-1)
}
fib(4)
```

JS101 壓平陣列

```js
function flatten(arr) {
  var result = [];
  for (var i = 0; i <= arr.length; i++) {
    if (Array.isArray(arr[i])) {
      var flatArr = flatten(arr[i]);
      flatArr.forEach((x) => result.push(x));
    } else {
      result.push(arr[i]);
    }
  }
  return result;
}
```

## 小技巧 - 大問題切成小問題

當問題卡關時，可以先用函式填空，把遇到的問題用函式分解到最小，就可以把解題大綱列出來了。

```js
// 印出 1~100 的偶數
function print1To100() {
  for (var i = 1; i <= 100; i++) {
    logEven(i);
  }
}
function logEven(num) {
  if (num % 2 === 0) {
    console.log(num);
  }
}
print1To100(); //呼叫
```

## Source

[All] 此篇文章 Lidemy JS101 的筆記，內容及圖片大部分取自上課影片
