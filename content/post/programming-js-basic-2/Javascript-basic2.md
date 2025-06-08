---
title: "[Javascript] 條件句、迴圈"
tags:
  - Javascript
categories:
  - Programming
date: 2020-06-17 22:23:00
slug: js-basic-2
---

## if/else statement

使用 if/else 來判斷狀況，用法如下：

### if...

可加入運算於判斷條件句內

```js
var num = 13;
if (num % 5 === 0) {
  console.log("num is multiple of five");
}
```

<!--more-->

### if...else

判斷如果 if 相符做什麼事，不相符則則做 else 區塊。

```js
var score = 70;
if (score >= 60 && score < 100) {
  console.log("pass");
} else {
  console.log("failed");
}
```

### if...else if...else

多條件判斷，先判斷 if、再判斷 else if(可以多個 else if 判斷)，最後都不相符的話執行 else 區塊。

```js
if (score >= 60 && score < 100) {
  console.log("pass");
} else if (score === 100) {
  console.log("excellent");
} else {
  console.log("failed");
}
```

### 補充

1. 小條件通常都用 () 括起來；區塊通常都用 {} 括起來
2. num % 5 === 0 　這個判斷 num 是否為 5 的倍數的判斷句，可以改寫成 !(num % 5)，但 num % 5 === 0 會比較直觀，如果無關乎效能的話．建議使用比較好懂得語句去寫 code。

## switch case

當判斷的條件非常多的時候，除了使用 else if 外，還可以使用 switch case 取代。

```js
var g = 1;
switch (month) {
  case 1:
    console.log("Jan");
    break;
  case 2:
    console.log("Feb");
    break;
  case 3:
  case 4:
    console.log("Mar");
    break;
  default:
    console.log("wrong month");
}
```

### 須留意

- <font color=darkred>在每個 case 結尾需加入 `break`</font>，當符合 case 後才會跳出，否則會把每個 case 都執行一遍
- 如果上述 case 都沒符合的話，會執行 `default`
- 可以把兩個 case 放一起做同一件事，例如上面範例的 case 3 & case 4，任一個符合都會執行 console.log('Mar')

### 小技巧

如果以這個範例來說的話，用上面寫法還是很冗，可以把它簡化成使用陣列來判斷變數內容。

```js
var month = 3;
var arrMonth = ["Jan", "Feb", "Mar", "Apr", "May"];
console.log(arrMonth[month - 1]); //找到 index 就是答案
```

所以可以依照需求去使用不同的寫法，看哪個最適合、或最簡便就用哪個。

### 同場加映小技巧

```js
var score = 60;
var isPass = false;
if (score >= 60) {
  isPass = true;
} else {
  isPass = false;
}
```

可簡化成以下，可讀性高又簡潔的寫法。

```js
var score = 60;
var isPass = score >= 60;
```

## 三元運算子 Ternary

`[condition] ? A : B`，condition 為成立的話執行 A；false 的話執行 B。

```js
console.log(10 > 5 ? "bigger" : "smaller");

var score = 60;
var msg = score >= 60 ? "pass" : "failed";
```

也可以加上巢狀判斷，但可讀性就會降低，所以不推薦寫。

```js
var msg = score >= 60 ? (score === 100 ? "excellent" : "pass") : "failed";
```

## 迴圈

定義

- 重複做一樣的事情
- 通常有終止條件
- 沒有終止條件的迴圈為`無窮迴圈`

### do...while...

do-while 迴圈至少會執行第一次。

```js
var i = 1;
do {
  console.log(i);
  i++;
} while (i <= 100);
console.log("i=", i); //i=101
```

當 while () 理的條件為 true 的話才會進入迴圈，false 的話會跳出迴圈。

#### break

或是可以把跳出迴圈的判斷寫在 do 裡面，使用 `break` 跳出迴圈。

```js
var i = 1;
do {
  console.log(i);
  i++;
  if (i > 100) {
    break;
  }
} while (true);
```

#### continue

可以在 do 裡面加上 `continue` 做另一層判斷以接進行下一圈。

```js
var i = 1;
do {
  if (i % 2 === 1) {
    continue;
  }
  console.log(i);
  i++;
} while (i <= 10);
```

上面印出的結果如下，因為當 i 為奇數時（% 2 === 1），會執行 continue 跳到 while 判斷進行到下一個個迴圈。

```
2
4
6
8
10
```

### while...

while 迴圈會先判斷，才會進入迴圈。通常較多使用 while 迴圈，依實際狀況而定。

```js
var i = 1; //初始值
while (i <= 100) {
  //終止條件
  console.log(i++); //i++, i每一圈要做的事情
}
```

while 同樣也可以使用上面提到的 `break` 以及 `continue`。

### for...loop

從上述 while 的例子中註解寫到的迴圈基本的元素，如果用 for-loop 形式表示如下：  
`for(初始值 i ; i 的終止條件; i 每一圈要做的事情){}`

```js
for (var i = 1; i <= 100; i++) {
  console.log(i);
}
```

執行的順序是 i=1、i 是否小於等於 100、console.log(i)、i++。  
for 同樣也可以使用上面提到的 `break` 以及 `continue`。  
for-loop 通常使用在已知迴圈長度的情況下，例如陣列。

#### 與陣列的搭配使用

使用 for loop 示範，也可以用 while / do-while。

```js
var scores = [1, 2, 3, 4, 5];
var sum = 0;
for (var i = 0; i < scores.length; i++) {
  sum += scores[i];
}
console.log("sum=", sum);
```

### 小技巧

利用 Chrome 的 debugger 可以實時觀察迴圈執行的變化，幫助釐清，可參考這篇[文章](https://ulahsieh.netlify.app/p/js-debugging-in-chrome/)。
