---
title: "[Javascript] runtime、運算子、變數"
tags:
  - Javascript
categories:
  - Programming
date: 2020-06-16 19:03:00
slug: js-basic-1
---

## Javascript Runtime

Javascript 需要有執行環境（Runtime）才能執行，最早期只能跑在瀏覽器上面，我們會透過 Javasrcipt 去操控瀏覽器畫面上的東西，例如表單驗證… 等等。  
但 **Node.js** 誕生後，就可以當成是 JS 除了瀏覽器外的 runtime，下載 Node.js 後便可以直接在 CMD 內下指令

`$ node < fileName.js >`

<!--more-->

不過兩個執行環境可以操控的東西不是完全一樣的，例如瀏覽器提供了 JS 操控文件的 API `document`，Node.js 提供 JS 操控檔案的 API `fs` module，這兩個 API 對於另外一個 runtime 都沒有。 但還是有共同都有的，例如 `console`，但呈現方式不同，瀏覽器的 console 如下圖，Node.js 的 console 就是 cmd 本身。

![](https://imgur.com/JO9wNhB.png)

## 運算子的種類

### 算數運算

| 運算 | 運算子 |
| :--: | :----: |
|  加  |   +    |
|  減  |   -    |
|  乘  |   \*   |
|  除  |   /    |
| 餘數 |   %    |
| 遞增 |   ++   |
| 遞減 |   --   |

#### i++ 跟 ++i 的差別

把 ++ 放後面代表 ++ 為`後運算`，如

```js=
var i=0;
console.log(i++ && 30);
```

會先執行 console.log(i && 30)，再執行 i + 1。

### 指派

|   運算   | 運算子 |            示例            |
| :------: | :----: | :------------------------: |
|  賦予值  |   =    | i = i + 1，將 i+1 指派回 i |
| 加法指派 |   +=   | `i += 1` 意即 `i = i + 1`  |

以此類推。

### 邏輯運算

| 運算 |    運算子    |
| :--: | :----------: |
| AND  |      &&      |
|  OR  | &#124;&#124; |
| NOT  |      !       |

#### 邏輯的短路性質

`3 || 10` 回傳 `3`；`0 || 10` 回傳 `10`。
第一個運算元是 true 的話，就回傳第一個參數（短路），否則傳第二個。  
`3 && 10` 回傳 `10`；`0 && 10` 回傳 `0`。 第一個運算元是 true 的話，由第二個決定，第一個運算元是 false 的話，直接回傳第一個 false（短路）。  
<font color=darkred>p.s. 在 Javascript 中，`0`、`false` 都代表 false。</font>

### 位元運算

將運算元**變成二進位**後，對每個位元(bit)做操作。

| 運算 | 運算子 |   　示例   | 結果 | 補充                                                    |
| :--: | :----: | :--------: | :--: | ------------------------------------------------------- |
| 左移 |   <<   |   8 << 1   |  16  | 左移就是乘以二                                          |
| 右移 |   >>   | 1024 >> 1  | 512  | 右移就是除以二                                          |
| AND  |   &    |   10 & 1   |  1   | `偶數 & 1` 會等於 1；`奇數 & 1` 會等於 0                |
|  OR  | &#124; | 2 &#124; 9 |  11  | `1 &#124; 1 == 1`、`1 &#124; 0 == 1`、`0 &#124; 0 == 0` |
| NOR  |   ^    |   3 ^ 10   |  9   | `0 ^ 0` 以及 `1 ^ 1` 等於 0，其他等於 1                 |
| NOT  |   ~    |     ~1     |  -2  | 00000001 變成 11111110                                  |

因為二進位是電腦最原始的表示方式，<font color=darkred>所以以二進位去運算，速度會比一般數學運算還快</font>，例如 `n << 1` 會比 `n * 2` 還要來得快，以 `n & 1` 判斷奇偶會比 `n % 2 == 0` 還快。

---

## 變數

### 變數宣告

使用 `var` 宣告變數，把它想成創造一個箱子存放東西。

```js
var test = 1; //宣告一個變數 test 並賦予值 1
```

### 命名規範

- 不能以數字開頭命名
- 大小寫不同視為不同變數
- 如果沒有宣告就使用變數的話會出錯並顯示 `undefined`
- 可以直接拿變數來做運算 (+, &&, ||, &, | ...)
- 命名變數要有語意，有兩種方式，且選擇一種後就要統一。
  - 底線式： api_response
  - 駝峰式：apiResponse

### 變數型態

七種原始型態 (primitive type)

- null
- undefined
- string
- number
- boolean
- symbol (ES6)

其他都是 Object 物件

- array
- object
- function
- date
- ...

使用 `typeof` 可以查看變數的型態：

![](https://imgur.com/GBOSvdZ.png)

所以使用 typeof 陣列會回傳他的底層型態- object

![](https://imgur.com/QgrSadW.png)

想確定變數是否為 array 時可以使用 `arr.isARRAY([])`。

另外使用 typeof null 會出先 Object，是 javascripy 的 bug，不須理會

### 陣列

如果變數是一個個不同的箱子的話，那麼陣列就是很多個差不多的箱子，放**同樣類型**的東西。

- 陣列的第一個元素索引式從 `0` 開始。
- 陣列在宣告的時候就可以給初始值。`var arr=[1, 3, 5, 7]`
- 要取得陣列的長度可以用 `length`，如 `console.log(arr.length)`，所以 `arr[arr.length-1]` 可以取得最後一個元素的值
- 要往陣列塞新的元素的話可以用 `push()` 方法，如

```js
arr.push(9);
console.log(arr); //[1, 3, 5, 7, 9]
```

### 物件 object

物件可以有多個屬性，屬性是 `key-value` 的映射。

```js
var student = [];
var peter = {
  name: "peter",
  score: 100,
  phone: "1234567",
};
student.push(peter); //把 peter 這個物件加入 student 陣列中
console.log(student[0].score); // 100
console.log(peter.name); //peter
console.log(peter["name"]); //peter
```

`arrayName.push(sth)` 可以往陣列後面塞元素。  
使用 `物件.屬性` 可以取得物件的屬性值；  
使用`物件['屬性']`，也可以取得物件的屬性值。  
這邊說明一下第二種方式主要是用來把 `[]` 中的屬性代入變數使用。例如

```js
var key = "name";
console.log(peter[key]);
```

#### 物件裡面可以放陣列、物件、或是函數

```js
var peter = {
    name : 'peter',
    score : [80, 100, 70, 100]
    phone: '1234567',
    contact: {
        name: 'nick',
        phone: '7654321'
    },
    sayhi: funciton(){
        //do sth.
    }
}
console.log(peter.father.name)
console.log(peter['father']['name'])
```

### 變數運算陷阱

#### 不同型態相加

同時有字串跟數字做相加的話，會變成兩個字串相加

```js
var a = 10;
var b = "20";
console.log(a + b); //1020
```

字串轉數字可以使用：

1. `Number(str)`
2. `parseInt(str,10)`，第二的參數代表十進位。

### 浮點數誤差

```js=
var a = 0.1 + 0.2
console.log(a == o.3) //false
console.log(a) //0.3000000000000004
```

原因是電腦在存小數的時候沒辦法存得這麼精準。

### == 和 === 的差別

```js=
console.log(0 == '0') //true, 判斷相等不判斷型態
console.log(0 === '0') // false, 判斷相等也需判斷型態
```

有時候只有兩個等號可能會有一些注意到的型態問題導致有 Bug，**所以建議永遠都用 === 三個等號做判斷**。

### 從 object 的等號真正理解變數

```js
console.log(100 === 100); //true
console.log([] === []); //false
console.log([1] === [1]); //false
console.log({} === {}); //false
console.log({ a: 1 } === { a: 1 }); //false
```

造成以上陣列或物件比較結果為 false 的原因是，javascript 底層真正在儲存物件的內容時是儲存該物件存放的 **記憶體位置**，所以指向不同記憶體，相比的結果就不同。

![](https://imgur.com/9Rxoz0m.png)

兩個變數指向**同一個物件**，兩變數去做比較就會相同，將物件內的屬性改值，兩個變數也都會改到。

```js
var obj1 = {
  a: 1,
};
var obj2 = obj1;
obj2.a = 2;
console.log(obj2 === obj1); //true
console.log(obj1.a); //2
```

![](https://imgur.com/UIHNws3.png)

![](https://imgur.com/86A8sXv.png)

如果指派另一個新的物件給變數 obj2，則會新建另一個記憶體位置存放。此時 obj1 就**不等於** obj2 了。

```
obj2={b:1}
console.log(obj2 === obj1) //false
```

![](https://imgur.com/tEqfcJK.png)

陣列的底層儲存也跟物件同理！

## Source

[All] 此篇文章 Lidemy JS101 的筆記，內容及圖片大部分取自上課影片
