---
title: "[CSS] 簡介- Selector & Selector Specificity"
categories:
  - Programming
tags: ["CSS"]
date: 2020-07-15 22:45:00
slug: "css-introduction-selector"
---

## CSS

Cascading Style Sheets 階層式樣式表，是一種用來為結構化文件（如 HTML 文件或 XML 應用）添加樣式（字型、間距和顏色等）的語言。

<!--more-->

## 幾種加入 CSS 的方式

1. inline style
   寫在 HTML 標籤中 (inline style)
   例如：

![](https://imgur.com/70G2pbh.png)

但比較不建議，因為會跟 HTML 混在一起，不便修改及維護。

2. `<style>`
   在 `<head>` 標籤區塊中加上 `<style>` 標籤

```html
<head>
  <style>
    div {
      background: red;
    }
  </style>
</head>
```

其中 `div` 是 selector，`background: red` 則是 property 及 value。

![](https://imgur.com/D3IiYSG.png)

但仍然不建議使用此方法，因為當 html 標籤一多，style 區塊中就要有許多 selector，也是不便於修改及維護。

3. `<link>`
   將 CSS 獨立於單獨的檔案中，再從 HTML 中使用 `<link>` 指定 css 檔進來。

```html
<head>
  <link rel="stylesheet" href="./style.css" />
</head>
```

```css=
/* style.css */
div {
    background : grey;
}
```

![](https://imgur.com/Sk5WoUv.png)

4. `@import`
   `@import` 語法有兩種：

```
<style>
  @import "style.css";
  @import url("style.css");
</style>
```

`@import` 是 CSS 中的一個 @ 規則，必須**先於**所有其他類型的規則 (@charset 規則除外)，須寫在 `<style>` 標籤裡。

**非模塊化開發時盡量不要用 @import**
使用 @import 會先導入 html 後才會加載 CSS 文件。

- 非模組化開發  
  正常開發時，所有的 CSS 文件都需要引入。如果在某個 CSS 文件中使用了 @import，瀏覽器要先下載使用了 @import 的 CSS，解析完後發現有另外的 CSS 文件需要下載，再去下載、解析，增加了用戶的等待時間。
  如果 CSS 內容不多，可以合並到一個文件里，減少請求次數。對於多個獨立的 CSS 文件，最好直接用 link 標簽加載。

- 模組化開發  
  在用 webpack 等工具開發時，會合併 CSS 文件。如果 CSS 文件相互之間有依賴，可以直接用也只能用 @import 引入，最後構建好的文件會合併文件，不會出現 @import。

{{< notice warning >}}
建議使用 `<link>` 方式引入， HTML 及 CSS 檔案彼此獨立，可以針對單一的需求做修改而不影響另一個，且

1. CSS 檔案加載與 HTML 檔案同時，網頁效能較佳。
2. 載入 javascript 進行 DOM 操作只能使用 link 導入
   {{< /notice >}}

---

## CSS 語法

![](https://imgur.com/0JwAeMX.png)

- Selector 選擇器，用來選擇 HTML 的元素。
- { } 大括號包圍了一個宣告區域，任何寫在這個區域裡的設定，會對 HTML 中符合選擇器的元素起作用。
- Property & Value：在圖例中，我們宣告選擇器 `<h1>` 的 color 屬性的值是藍色、font-size 屬性的值是 12px。
- 多條宣告用分號 ( ; ) 隔開，可以使用空格或換行增加可讀性。
- 注釋使用 `/* */`。

## Selector 的種類

### Universal Selector

使用 `*` ，代表選擇**所有的** html 元素。

```css=
* {
    color : orange;
}
```

### Element Selector

使用特定的標籤，代表選擇所有指定標籤。

```css=
div {
    color : orange;
}
```

### ID Selector

在一個 HTML 文件中單一 ID 只能被使用**一次**。選擇器以 `#` 開頭。

```html
<div id="a">test</div>
```

<br>

```css
#a {
  color: red;
}
```

### Class Selector

當你有相同的東西要長得一樣時就可以把他們都命名為同一個 class。選擇器以 `.` 開頭。

```html
<div class="sty1">hello</div>
<div class="sty1">world</div>
```

<br>

```css
.sty1 {
  color: red;
}
```

<br>

{{< notice info >}}
至於何時要用 ID 、何時要用 Class，並沒有絕對的規則。但是大多的時候，Class 選擇器的使用頻率較高，因為較靈活，目的為了將幾個元素從其他元素中獨立出來做改動。但是當你要用 Javascript 的 GetElementByID 函數時，你就應該要用 ID 選擇器，因為 ID 選擇器可以被 Javascript 中的 GetElementByID 函數所運用。
{{< /notice >}}

### 同時符合

把 selector 連在一起，可以表示同時符合多種條件的選擇器。

```html
<div class="sty1">hello world</div>
<div>hello html</div>
<span class="sty1"> hello css </span>
<div class="sty1 bg-yellow">hello js</div>
```

<br>

```css
/* 同時符合是　div 標籤以及 .sty1 class 的元素 */
div.sty1 {
  color: red;
}
/* 同時符合是　div 標籤以及 .sty1 class 以及 bg-yellow class 的元素 */
div.sty1.bg-yellow {
  color: red;
  background: yellow;
}
```

### 選取底下的元素

```html
<div class="lv1">
  level 1
  <div>
    level 2
    <div class="bg-red">level 3</div>
  </div>
  <div class="bg-red">level 2-1</div>
</div>
```

#### 使用 `>` 選取左邊元素的下一層右邊元素

```css
/*選擇 .lv1 下一層的所有的 .bg-red*/
.lv1 > .bg-red {
  color: orange;
}
```

![](https://imgur.com/CfTiiJj.png)

#### 使用 ` ` 選取左邊元素下一層內的所有符合右邊的元素

```css
/*選擇 .lv1 內的所有的 .bg-red*/
.lv1 .bg-red {
  color: orange;
}
```

![](https://imgur.com/CfTiiJj.png)

### 選取旁邊（同一層）的元素

```html
<div class="bg-red">div1</div>
<div>div2</div>
<div class="bg-red">div3</div>
<div class="bg-red">div4</div>
<span>123</span>
<div>456</div>
<div>789</div>
```

#### `+` 右邊一個

```css
/* 選 bg-red (第三個 div) 旁邊的 bg-red (第四個 div) 標籤 */
.bg-red + .bg-red {
  background: red;
}
/* 選 span 旁邊的 div */
span + div {
  background: yellow;
}
```

![](https://imgur.com/a3zm31F.png)

#### `~` 右邊全部的

```css
/* 選 bg-red (第一個 div) 後面所有的 bg-red 標籤 */
.bg-red ~ .bg-red {
  background: red;
}
/* 選 span 後面所有的 div */
span ~ div {
  background: yellow;
}
```

![](https://imgur.com/pMntnXU.png)

這個語法很適合用在排版，通常想要將一排的物件平均均分，第一個通常要靠邊界對齊不會動，在後面每個要間格 80px 的話只需要使用 `~` 選取。

![](https://imgur.com/qHA3r2y.png)

### Pseudo-classes 偽類

#### :hover

當游標移動到指定 selector 上面時要做什麼樣的變化。

```css
div:hover {
  background: red;
}
```

![](https://imgur.com/VzN9A6g.gif)

#### nth-child

```html
<div class="wrapper">
  <div>div1</div>
  <div>div2</div>
  <div>div3</div>
  <div>div4</div>
  <div>div5</div>
  <div>div6</div>
</div>
```

<br>

```css
.wrapper div:first-child {
  /* 選 wrapper class 下的第一個元素同時是 div */
  background: red;
}

.wrapper div:last-child {
  /* 選 wrapper class 下的最後一個元素同時是 div */
  background: orange;
}

.wrapper div:nth-child(3) {
  /* 選 wrapper class 下的第三個元素同時是 div */
  background: blue;
}

.wrapper div:nth-child(odd) {
  /* 選 wrapper class 下的奇數位看同時是 div */
  background: yellow;
}

.wrapper div:nth-child(3n) {
  /* 選 wrapper class 下的三的倍數位看同時是 div */
  background: green;
}

.wrapper div:nth-child(3n + 1) {
  /* 選 wrapper class 下的三的倍數加一位看同時是 div */
  background: green;
}
```

- 其他的 Pseudo-classes

```css
:link /*連結平常的樣式*/
:visited /*連結查閱後的樣式*/
:active /*滑鼠按下的樣式*/
:focus /*目標為焦點的樣式*/
:lang(E) /*當語言為E的樣式*/
:first-child /*第一個元素的樣式*/
```

更多偽類別請參考：[Pseudo-classes](https://www.w3schools.com/css/css_pseudo_classes.asp)

{{< notice info >}}
補充  
因為 Pseudo-classes 通常都需要使用者觸發才會有樣式的變化，會較難 debug。下面展示了如何使用開發人員工具讓我們不須使用滑鼠也能測試 Pseudo-classes。
{{< /notice >}}

![](https://imgur.com/ZiWXe25.png)

### Pseudo-element 偽元素

可以選到某個元素的某個部分，為了與 pseudo-class 區分，以兩個冒號`::`來表示。

```html
<div class="wrapper">1000</div>
```

![](https://imgur.com/gr3x9Ju.png)

#### Before

```css
.wrapper::before {
  content: "$";
  color: red;
}
```

![](https://imgur.com/SEB98dY.png)

#### After

```css
.wrapper::after {
  content: "NTD";
  color: red;
}
```

![](https://imgur.com/xcr6JQh.png)

### 在 content 放入 selector 的 property value

```css
.wrapper::after {
  content: attr(class);
  color: red;
}
```

![](https://imgur.com/2j8aZJ6.png)

```html=
<div class="wrapper" property="value">
  1000
</div>
```

<br>

```css
.wrapper::after {
  content: attr(value);
  color: red;
}
```

![](https://imgur.com/SnnyAAs.png)

---

## CSS Selector 的權重

所謂的權重就是指 css 的優先權:

- 相同權重但是後寫的 css 可以覆蓋先寫的 css
- 當兩個選擇器同時作用在一個元素，權重高的優先生效

權重大小(可以想成愈詳細的愈贏)

{{< notice warning >}}
!important > inline style > ID > Class/psuedo-class(偽類)/attribute（屬性選擇器） > Element
{{< /notice >}}

### !important

!important 的權重最高，可以蓋過所有的權重，建議不要隨便使用。

```html
<div style="color=blue">hello</div>
```

<br>

```css
div {
    color: red;!important
}
```

最後 hello 的樣式會是採用 !important 的紅字。

一個可愛的 CSS 權重圖：

![](https://imgur.com/LtsCyMr.png)

## Reference

[1]https://www.w3schools.com/css/css_syntax.asp
[2]https://ithelp.ithome.com.tw/questions/10183484
[3]https://ithelp.ithome.com.tw/articles/10196454
[4]http://www.standardista.com/css3/css-specificity/
[5]https://www.itdaan.com/tw/78e32ccfaac206f9b1aab87e586f3b0a
