---
title: 新手刷題前應該知道的事
categories:
  - Programming
date: 2020-06-19 21:42:00
slug: lidimy-alg101-note
---

這篇文章是 Lidemy Mentor Program 第二周觀看 ALG101 先別急著寫 Leetcode 的筆記。

## 新手寫題目怎麼開始

1. 先想解法
2. 在把解法轉成程式碼
   而不是著急的直接邊寫程式碼邊解，要先有一個架構之後再去想怎麼轉換成 code。
   <!--more-->
   舉例：印出 1~100 的奇數
3. 想解法
   - 印出一到一百
   - 印的時候判斷是不是奇數
4. 把解法換成程式碼

利用虛擬碼 psedu code 來表示程式碼

![](https://imgur.com/NR7Hxwo.png)

{{< notice warning >}}
縮排的重要性！縮牌等於一個區塊，例如 if-end if 代表一個區塊，大大的增加程式可讀性。
{{< /notice >}}

![](https://imgur.com/a9znaI2.png)

需要練習當看到程式碼時，腦中要能知道程式的流程。
可以想想看如果把 max 的初始值設成 arr[0] 的話，arr 在 [7, 5] 的狀況下答案會是對的嗎?

## 善用 log & debugger

用 Debugger 可以讓程式一行一行的跑 code，方便新手了解程式執行流程並找出哪一行有錯。
關於 Chrome Devtool Debbuger 的使用之前就有特別記錄成一篇[文章](https://ulahsieh.netlify.app/p/js-debugging-in-chrome/)。

## 範圍很重要

因為不同輸入範圍代表不同限制，同一個題目可能輸入範圍不一樣，解法就會不同。解題時可能會碰到的三種限制：

1. 空間限制

```
一般程式語言中的 int： 4 bytes；double：8 bytes
JS 中的 Number：8 bytes
```

當今天程式要使用到一百萬個數字，則大約需要 7.6 MB；十億個數字，則大約需要 7.4 GB 的空間，若要排序一百萬個數字可能還可以宣告陣列來存，但如果換成十億筆的話就需要使用其他方式，比如說先放在檔案，再取出比較。 2. 時間限制 3. 型態限制
不同型態可以儲存的範圍不同，例如 Number.MAX_SAFE_INTEGER 是 Javascript 可以儲存的最大安全數值，如果超過此數，就沒辦法保證正確性，如下範例：

![](https://imgur.com/yRtYnpJ.png)

另外還有浮點數的精準度問題

![](https://imgur.com/GzPTLN7.png)

## 遇到困難時

請利用函式填空法以及簡化法，先避開題目的細節，把架構先做出來。當你把架構寫出來後，會發現剩下的東西其實沒那麼困難。如下圖範例，多了函式，整體架構會看起來更清楚。

![](https://imgur.com/6bWA65a.png)

## 一些上課學到的小技巧

1.

```js
function total() {
  if (total % 10 === 0) {
    return true;
  } else {
    return false;
  }
}
```

可以轉換成 `return total % 10 === 0` 一行

2. key-value 對應關係除了使用 switch case 以及 if-else 判斷外，可以使用`物件.屬性(key)`做存取。

```js
function getValue(s) {
  var mapping = {
    A: 10,
    B: 11,
    C: 12,
    D: 13,
  };
  return mapping[s];
}
console.log(getValue(A)); //輸出 10
```

3. callback function 與 `=>` 的轉換

```js
function main() {
  let arr= [1,2,3,4,5,6]
  let target = 3
  let newArr = filter(arr, function(x) {
    return x !== target
  })
  // 可以寫成 filter(arr, x => x !== target)
  })
  console.log(newArr)
}

function filter(arr, callback) {
  let result = []
  for(let i=0; i<arr.length; i++) {
    if(callback(arr[i])) {
      result.push(arr[i])
    }
  }
  return result
}
```

4. 使用`.map` 將陣列中元素換成 Number 型態

```
var arr = ['1', '2', '3'].map(Number)
```

一行結束!!

5. `return <判斷式>` 的簡寫

```
function isPositive(n) {
	return n > 0
}
```

判斷 n 是否為正的 function ，回傳 true / false 可以直接在 return 使用判斷式判斷並回傳。

## 程式效率

使用**時間複雜度** Big O 符號來表示執行所需步驟與資料量 n 的關聯：
O(n)：執行所需步驟與 n 成正比
O(2^n)：執行所需步驟與 2^n 成正比
O(1):執行所需步驟與 n 沒有關係

![](https://imgur.com/Vivt0gQ.png)
[1]
相對的是空間複雜度，通常兩者不可兼得。
關於時間與空間複雜度，前幾篇計概的[文章](https://ulahsieh.netlify.app/p/lidimy-cs101-note/)有稍微提到。

## Source

[1] http://bigocheatsheet.com/
[All] 此篇文章 Lidemy <ALG101 先別急著寫 Leetcode> 的筆記，內容及圖片大部分取自上課影片
