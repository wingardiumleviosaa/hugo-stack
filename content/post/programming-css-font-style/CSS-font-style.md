---
title: "[CSS] 文字樣式 & 文字斷行"
categories:
  - Programming
tags: ["CSS"]
date: 2020-07-20 22:47:00
slug: "css-font"
---

## 文字相關的 CSS 屬性

### color

文字顏色，值可以是顏色敘述如 `red`、[色碼表](https://www.ifreesite.com/color/) `#FF0000`，或是 `rgb(255,0 , 0)`。另外 `rgba(255, 0, 0, 0.5)` 的 a 表示透明度，值從零到一。

<!--more-->

### font-size

文字大小。

### font-weight

字的粗細，值有 `normal, bold, lighter, bolder` 或是 100 ~ 900 的整百數值。常見的字重數值大致對應的字重描述詞語：

```
100 - Thin
200 - Extra Light (Ultra Light)
300 - Light
400 - Regular (Normal、Book、Roman)
500 - Medium
600 - Semi Bold (Demi Bold)
700 - Bold
800 - Extra Bold (Ultra Bold)
900 - Black (Heavy)
```

### font-family

字體。可以接多個字體，如果字體名稱中有空白，就必須用 `''` 或 `""` 括起來。
有兩種類型的字體系列名稱：

- family-name：指定字體，如 `times`、`courier`、`arial`。
- generic-family：通用字體，如 `serif`、`sans-serif`、`cursive`、`fantasy`、`monospace`。  
  當瀏覽器載入網頁樣式時，會從 font-family 的第一個字體開始判斷，如果沒有對應的字體，就會採用下一種字體，如果到最後都沒有可用的字體，就採用電腦預設字體，一般來說定義 font-family 時，會將泛用字放在最後方，此時就可以透過 generic-family 來指定哪種預設字體。

### letter-spacing

字的間距。值可以是 `normal`(按照當前字體的正常間距確定)、`3px`、`0.3em`。

### line-height

行距，兩行文字中間的高度。值可以是 `normal`(按照當前字體的正常行距確定)、`3px`、`0.3em`。

### text-align

水平對齊，常見屬性有：

- text-align:left; 向左對齊
- text-align:right; 向右對齊
- text-align:center; 置中
- text-align:justify; 使左右對齊本文，通常用於整篇文章
- text-align:inherit; 繼承父元素的 text-align 屬性

### text-decoration

可能的屬性值有 `none/ underline/ overline/ line-through`，分別是 `無作用/ 底線/ 上划線/ 刪除線`。
`none` 可以使用在去除超連結自動加上的底線，使用 text-decoration: none 可將底線去除。

---

### 利用 line-hight 來達成垂直置中

把行距與元素高度調成一致來達成垂直置中，僅適用於文字只有一行時。

![](https://imgur.com/AC3wncG.png)

### 利用 padding 來達成垂直置中

或是單純利用 padding 把元素的邊框與內容撐開，可用於多行。

![](https://imgur.com/Lmw58fy.png)

---

## 文字斷行

### word-break

設定字串斷行的時機，屬性值常用的有三個：

- normal：CJK(即 Chinese/Japanese/Koean，中日韓) 文本插入斷行，其它文本不插入斷行
- break-all：所有文本都會插入斷行
- keep-all：所有文本都不會插入斷行

![](https://imgur.com/DQFnW1A.png)

### word-wrap

基本上和 word-break 一樣是設定字串換行，常用的屬性值：
normal：不會在非 CJK 單詞中插入斷行
break-word: 會在單詞中插入斷行避免溢出

![](https://imgur.com/anF6t5j.png)

### word-break & word-wrap 的差別

![](https://imgur.com/pTdP4xW.png)

word-break: break-all 值如其名，斷開一切，利用上每一塊可以利用的空間來塞下文本，避免鋪張浪費；而 word-wrap: word-break 則收斂許多，假如一行文字有可以換行的點，如標點、CJK 文本等，那麼就不會在英文單詞或者字符中插入斷行了，不過從顯示效果來說的話則容易一塊兒密集、一塊兒空白，較不美觀。

### white-space

在 word-break, word-wrap 什麼都沒設的情況下，white-space 就有預設值 normal，會幫我們自動換行。如果要取消換行的話則屬性值輸入 `nowrap`.

![](https://imgur.com/xvTH4VL.png)

---

### overflow

針對超出元素範圍的內容作處理，屬性值有：

- visible：就給他超出去，default 值。
- hidden：把超出的部分隱藏
- scroll：多出 scroll bar 捲動多出來的部分
- auto：由瀏覽器自動選擇，一旦超出範圍則自動變 scroll bar
- inherit：繼承父元素的屬性值

### text-overflow

針對超出元素範圍的字串做處理，屬性值有：

- ellipsis：用 `...` 來取代多餘的字串
- clipe：將超出範圍的字切斷
- 字串：使用指定字串來取代多餘的字串

{{< notice warning >}}
請注意

1. 要使用 text-overflow: ellipsis 的先決條件是要先設置 white-space:wrap & overflow:hidden;
2. overflow 用的範圍比較廣，text-overflow 就僅針對文字而已。
   {{< /notice >}}

![](https://imgur.com/gILIjIx.gif)

---

### 列表樣式

#### list-style-position

可以用的參數值有 `inside/ outside`，
inside 指項目符號在 `<li></li>` 標籤範圍之內。
outside 指項目符號在 `<li></li>` 標籤範圍之外，為預設值。

```html
<ul style="list-style-position:inside;">
  　
  <li style="border:1px #cccccc solid">Test list style position inside.</li>
  　
  <li style="border:1px #cccccc solid">Test list style position inside.</li>
</ul>
<ul style="list-style-position:outside;">
  　
  <li style="border:1px #cccccc solid">Test list style position outside.</li>
  　
  <li style="border:1px #cccccc solid">Test list style position outside.</li>
</ul>
```

<ul style="list-style-position:inside;">
　<li style="border:1px #cccccc solid">Test list style position inside.</li>
　<li style="border:1px #cccccc solid">Test list style position inside.</li>
</ul>
<ul style="list-style-position:outside;">
　<li style="border:1px #cccccc solid">Test list style position outside.</li>
　<li style="border:1px #cccccc solid">Test list style position outside.</li>
</ul>

#### list-style-type

常用的屬性值如下表：

<table><tr class="Line"><td bgcolor="#eeeeee" width="160"><b>參數</b><br></td><td bgcolor="#eeeeee"><b>定義</b><br></td></tr><tr class="Line"><td bgcolor="#eeeeee">none<br></td><td>不顯示符號<br></td></tr><tr class="Line"><td bgcolor="#eeeeee">disc<br></td><td>實心圓形<br></td></tr><tr class="Line"><td bgcolor="#eeeeee">circle<br></td><td>空心圓形<br></td></tr><tr class="Line"><td bgcolor="#eeeeee">square<br></td><td>實心正方形<br></td></tr><tr class="Line"><td bgcolor="#eeeeee">lower-alpha<br></td><td>小寫英文字母<br></td></tr><tr class="Line"><td bgcolor="#eeeeee">upper-alpha<br></td><td>大寫英文字母<br></td></tr><tr class="Line"><td bgcolor="#eeeeee">decimal</td><td>阿拉伯數字</td></tr><tr class="Line"><td bgcolor="#eeeeee">decimal-leading-zero</td><td>十進位制的阿拉伯數字，前方自動補零</td></tr></table>

#### list-style

可以直接使用 `list-style` 簡化表示上面兩個屬性，例如 `list-style: circle inside;`。

## Reference

[1]https://www.w3school.com.cn/
[2]http://www.webtech.tw/
[3]https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/655483/
