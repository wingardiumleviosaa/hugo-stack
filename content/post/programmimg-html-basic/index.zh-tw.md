---
title: "[HTML] 介紹 & 基礎標籤"
categories:
  - Programming
tags: ["HTML"]
date: 2020-07-12 21:26:00
slug: "html-introduction"
---

## 網頁是什麼

網頁背後就是一個有結構的純文字檔 html，靠瀏覽器渲染成我們現在看到的畫面。

<!--more-->

![](https://imgur.com/nv5JkQo.png)

## HTML

全名是 HyperText Markup Language 超文本標記語言，是一種用於建立網頁的標準標記語言，是由一堆成對的標籤組合而成的。

## 基本架構

```html=
<!DOCTYPE HTML>
<html>
  <head>
  </head>
  <body>
  </body>
</html>
```

### 撰寫規則

![](https://imgur.com/UQbNXRe.png)

1. HTML 文件裡的標籤需成對，以`<標籤名稱>`為開頭；`</標籤名稱>`為結尾。
2. 如果標籤沒有值要包裹的話，則可以直接使用 `<標籤名稱 />` 視為一組標籤。
3. 標籤內可擁屬性，`<標籤名稱 屬性名稱="屬性值">標籤內容<?標籤名稱>`，屬性可有零個或多個，以空格分開。
4. 註解文字使用 `<!--註解-->`。
5. HTML 標籤不分大小寫。

---

#### `<!DOCTYPE HTML>`

告知瀏覽器要使用最新版的 HTML 5 語法解讀文件，規定**必須放在第一行**，否則瀏覽器會認不得。

#### `<html></html>`

用來包裹整個 html 內含的程式碼，代表網頁的開始和結束。

#### `<head></head>`

用來放網頁的描述。主要用來告訴搜尋引擎這個網頁有什麼樣的內容、控制網頁與外部程式碼的連結 (script)、定義網頁使用的樣式等等 (css)。常用的標籤有：

- `<title>`: 網頁標題
- `<meta>` : 提供搜尋引擎關於網頁內容的簡介
- `<link>` : 網頁內與外部資源的關連
- `<base>` : 設定網頁內 URL 的預設目標
- `<style>` : 導入網頁樣式，如 CSS
- `<script>` : 導入 javascript 程式碼

```html
<head>
  <meta charset="utf-8" />
  <meta name="keywords" content="網頁關鍵字" />
  <meta name="description" content="網頁簡短描述" />
  <title>Hello!</title>
</head>
```

- `<meta charset="utf-8">`：告訴瀏覽器這個網頁所用的編碼方式為 utf-8。
- `<meta name="keywords"`：用來放置網頁關鍵字，可增進 SEO(Search Engine Optimization)，讓搜索引擎找到。
- `<meta name="description"`：用來寫網頁的簡短描述，可增進 SEO。
- `<title>Hello!</title>`：網頁的標題，顯示在書籤頁上。

---

#### `<body></body>`

網頁的內文，真正顯示的頁面內容放在 body 標籤內。

![](https://imgur.com/oGXkADV.png)

---

##### `heading & paragraph`

- `<h?></h>`：標題，其中 `? = 1 ~ 6`，由大到小 `<h1> 到 <h6>`。
- `<p>段落文字</p>`

![](https://imgur.com/Kj8pDfB.png)

- `<hr />`：文章分割線。

##### `pre & break`

在段落中`<p>`使用空行以及多個空白，都僅會被瀏覽器解析成一個空白，如下圖。

![](https://imgur.com/VCAO59k.png)

- `<pre>`：preformatted text，可將包裹的文字保持原先的格式。
- `<br>`：line break，換行，`<br>` 或是 `<br/>` 都可以。

![](https://imgur.com/Sh4udTP.png)

##### `div & span`

- `<div>` tag：division，使用 div tag 可以在網頁內創造不同的分組，可以直接使用 div 來加入 CSS 語法做排版。
- `<span>` tag：span 與 div 一樣都是用來分組，但不同的是使用 span 不會換行。

![](https://imgur.com/dL49EQF.png)

##### `img`

使用 `<img src="圖片網址" />` 來插入圖片，其中可以加入以下屬性。

- title：`<img title="圖片敘述" src="url">`，可在網頁中當游標移到圖片上時顯示描述。
- alt：`<img alt="image not found" src="url">`，當圖片無法顯示時，將以 alt 屬性值的文字取代顯示。

---

##### `list`

- `<ul>`： unordered list，無序清單
- `<ol>`： ordered list，有序清單
  - `<ol type='?'>`：其中 ? 可以是 `1 (default)`、`a`、`A`、`i`、`I`，表示序號的類型。
    - `<ol start='2'>`：指定序號起始位置。
- `<li>`： list item，清單項目

![](https://imgur.com/4KyfHRC.png)

##### `define list`

`<dl>` 定義了一個定義列表，通常用來描述一些術語定義，比如附錄裡的詞彙表，或用來顯示 key-value 這樣成對的鍵和值。

- `<dt>`：definition title，定義標題
- `<dd>`：definition description，定義描述

示例：

```html
<dl>
  <dt>Firefox</dt>
  <dd>
    A free, open source, cross-platform, graphical web browser developed by the
    Mozilla Corporation and hundreds of volunteers.
  </dd>

  <!-- other terms and definitions -->
</dl>
```

</br>

<table><tr><td bgcolor=#FAFAFA>
<dl>
  <dt>Firefox</dt>
  <dd>A free, open source, cross-platform, graphical web browser
      developed by the Mozilla Corporation and hundreds of volunteers.</dd>

  <!-- other terms and definitions -->
</dl>
</td></tr></table>

---

##### `table`

- `<table>`：表格
- `<tr>`：table row
- `<th>`：table header
- `<td>`：table cell

![](https://imgur.com/4tiOPPO.png)

使用以上標籤來製作表格內容，另外框線是 CSS 的工作。
可以直接在 `<table>` 標籤內加入屬性：

- width, height：設定整個表格的欄位的寬高。
- border：設定表格外框。
- cellspacing：表格欄位間的距離。
- cellpadding：表格欄位內容與邊框的間距，預設為 1。

![](https://imgur.com/pacz1D4.png)

- align：屬性值有 center, left, right 的對齊方式，可放在不同標籤內而針對不同的對象。
  - table 標籤：針對整個表格在整個畫面上的對齊
    - tr 標籤：針對一整個列的表格內容在表格內的對齊
    - td 標籤：指單一個表格的內容對齊
- valign：<font color="Crimson">只能放在 `<tr>` 或 `<td>` 標籤內</font>，屬性值有 top, middle, bottom。

- colspan：合併行欄位，屬性值為欲合併的欄位數。

```html
<table>
  <tr>
    <td colspan="3">合併</td>
  </tr>
  <tr>
    <td>1</td>
    <td>2</td>
    <td>3</td>
  </tr>
</table>
```

</br>

<table>
	<tr>
        <td colspan="3">合併</td>
    </tr>
    <tr>
    	<td>1</td>
        <td>2</td>
        <td>3</td>
    </tr>
</table>

- rowspan：合併列欄位，屬性值為欲合併的列位數。

```html
<table>
  <tr>
    <td rowspan="2">合併</td>
    <td>2</td>
    <td>3</td>
  </tr>
  <tr>
    <td>2</td>
    <td>3</td>
  </tr>
</table>
```

</br>

<table>
	<tr>
        <td rowspan="2">合併</td>
        <td>2</td>
        <td>3</td>
    </tr>
    <tr>
        <td>2</td>
        <td>3</td>
    </tr>
</table>

---

##### `anchor`

`<a>` 錨點，有兩個用途，連結到外面的位址或是連結到網頁本身內部位置。

- href：hypertext reference，`<a href="https://google.com">請點我連結到 google</a>`。
- target：`<a href="https://google.com" target="_self">請點我連結到 google</a>`，屬性值如果是`_self`，則會在當下頁面跳轉，屬性值如果是`_blank`，則會在另外的分頁開啟。
  連結到網頁本身則需要在標題標籤內加上 id 的屬性，如下圖：

![](https://imgur.com/bCz7kKF.gif)

---

##### `語意化元素`

![](https://imgur.com/RSjKkL0.png)

語意化元素 Semantic Elements，雖然在網頁上看起來不會有什麼變化，但是卻有他的意義，讓機器或開發人員更直覺的區別該區塊的作用。

- `<main>`：指網頁最主要的部份，可用 main 標籤包起來。
- `<nav>`：navigation 導覽列，至於網頁上方的選單。

![](https://imgur.com/CQ5xNz2.png)

- `<footer>`：通常放在網站最下方，例如版權宣告...。

![](https://imgur.com/3pwe9cZ.png)

更多的語意化標籤可以在 [w3schools](https://www.w3schools.com/html/html5_semantic_elements.asp) 上查到。

---

##### `iframe`

用來嵌入其他網頁，但很多大網站都會加入 `X-Frame-Option` 去禁止被嵌入。
下圖示範將部落格嵌入到網頁中，並設定寬為 100%，高為 250 像素。

```html
<iframe src="https://ulahsieh.github.io/" width="100%" height="250px"></iframe>
```

![](https://imgur.com/leC1UGj.png)

---

##### `form`

`<form>` 用於創建 HTML 表單，屬性值有下列幾種：

<table width="90%" border="0" cellspacing="0" cellpadding="10">
    <tr valign="top"> 
      <td width="32%">action = <i>url</i></td>
      <td width="68%">指定表單處理程式的 url。若不指定，則表單不會被處理。</td>
    </tr>
    <tr valign="top"> 
      <td width="32%">method = get | post </td>
      <td width="68%">指定何種送出表單資料的 HTTP 方式。可能值為 get（預設值）或 post。</td>
    </tr>
    <tr valign="top"> 
      <td width="32%">name = <i>string</i></td>
      <td width="68%">指定表單的名稱。客戶端程式（如 JavaScript）或伺服端程式（如 ASP 或 PHP）可以用這個名稱來存取表單的內容，因此，你應該用這一個屬性替每一個表單取名，以便利撰寫處理表單的程式。</td>
    </tr>
    <tr valign="top"> 
      <td width="32%">enctype = <i>content-type</i> </td>
      <td width="68%">當 method 屬性設成 post 時，這個屬性指定資料送出的格式。預設值為 application/x-www-form-urlencoded。一般而言，使用預設值即可，但是製作上傳檔案的表單時，這個屬性應該改為 
        multipart/form-data。 </td>
    </tr>
    <tr valign="top"> 
      <td width="32%">accept-charset =<i> charset-list</i></td>
      <td width="68%">指定表單資料的字元編碼格式。若不只一種格式的話，格式名稱之間必須用空白或逗號分開。預設值為 UNKNOWN。一般而言，使用預設值即可。</td>
    </tr>
    <tr valign="top"> 
      <td width="32%">accept = <i>content-type-list </i></td>
      <td width="68%">指定表單內容的格式。若不只一種格式的話，格式名稱之間必須用逗號分開。這個屬性常配合上傳檔案的功能，讓瀏覽器用其所指定的格式來篩選檔案。 
      </td>
    </tr>
</table>

###### `<input>`

需要搭配屬性 `type` 使用，常見的屬性值有：

- text，一般文字輸入框。
  - placeholder：輸入提示
- password，可自動將輸入的文字轉不可讀的點點。
- email，會自動驗證是否符合 email 格式（包含 ＠）。

![](https://imgur.com/W3ODwqT.png)

- date，可以自動產生出日期選擇器。
- radio，單選框，`<input type="radio" name="gender" id="male"/><label for="male">男</label>`
  - name：將選項**群組化**，使答案只能單選。
  - checked：默認選項
  - id & lable for：使得點選選項的文字也能作用，可提升使用者體驗。
- checkbox，複選框，用法同上，差別在選項可複選。
- submit，提交的按鈕，加上 value 屬性可以修改按鈕文字，點擊後會跳轉(執行) form 標籤中的 action 的 api。
- reset，重置表單域中的欄位內容。

###### `<select>`

是下拉式選單的最外層元件，具有的屬性如下：

- name = string，指定元件的名稱。
- size = number，指定同時顯示出的選項個數，預設值為 1。
- multiple，允許複選。

###### `<option>`

這個元件用來建立選單中的選項，必須擺在 `<SELECT>` 元件之中。它的屬性如下：

- selected：預選這一個選項。
- value = string：指定選項的代表值。
- label = string：指定選項的替代標籤文字。

```html
<form>
  <!--使用 div 隔開換行-->
  <div>姓名：<input type="text" placeholder="請輸入姓名" /></div>
  <div>密碼：<input type="password" /></div>
  <div>Email：<input type="email" /></div>
  <div>生日：<input type="date" /></div>
  <div>
    性別：<input type="radio" name="gender" id="male" checked="checked" /><label
      for="male"
      >男</label
    >
    <input type="radio" name="gender" id="female" /><label for="female"
      >女</label
    >
    <input type="radio" name="gender" id="other" /><label for="other"
      >其他</label
    >
  </div>
  <div>
    興趣：<input type="checkbox" id="sport" /><label for="sport">運動</label>
    <input type="checkbox" id="read" /><label for="read">看書</label>
    <input type="checkbox" id="music" /><label for="music">聽音樂</label>
    <input type="checkbox" id="drama" /><label for="drama">追劇</label> 　
  </div>
  <div>
    <input type="submit" value="送出表單" />
  </div>
</form>
```

</br>

<table><tr><td bgcolor=#FAFAFA>
<form>
  <div>
    姓名：<input type="text" placeholder="請輸入姓名" />
  </div>
  <div>
    密碼：<input type="password" />
  </div>
  <div>
    Email：<input type="email" />
  </div>
  <div>
    生日：<input type="date" />
  </div>
  <div>
    性別：<input type="radio" name="gender" id="male" checked="checked"/><label for="male">男</label>
    <input type="radio" name="gender" id="female"/><label for="female">女</label>
    <input type="radio" name="gender" id="other"/><label for="other">其他</label>
  </div>
  <div>居住地：
  	<select name="city" size="1">
  		<option value="taipei">台北市</option>
  		<option value="taichung">台中市</option>
  		<option value="Kao">高雄市</option>
  	</select>
  </div>
  <div>
    興趣：<input type="checkbox" id="sport"/><label for="sport">運動</label>
    <input type="checkbox" id="read"/><label for="read">看書</label>
    <input type="checkbox" id="music"/><label for="music">聽音樂</label>
    <input type="checkbox" id="drama"/><label for="drama">追劇</label>
　</div>
  <div>
    <input type="submit" value="送出表單"/>
    <input type="reset" value="清除"/>
  </div>
</form>
</td></tr></table>

### 在網頁中顯示標籤

使用跳脫字元（escape）

- `<` 由 `&lt;` 取代
- `>` 由 `&gt;` 取代
- `&` 由 `&amp;` 取代
- ` ` (空格) 由 `&nbsp;` 取代

## Reference

- http://notepad.yehyeh.net/Content/WebDesign/HTML/01/1.php
- https://www.w3schools.com/html/html5_semantic_elements.asp
- http://know.webhek.com/html5/html-dl-dt-dd.html
- https://www.cs.pu.edu.tw/~tsay/course/webprog/notes/form
- 此篇為觀看 Lidemy FE101 的筆記，部分內容取自上課影片
