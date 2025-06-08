---
title: "[CSS] overflow 應用"
categories:
  - Programming
tags: ["CSS"]
date: 2020-10-15 17:48:00
slug: "css-overflow"
---

## Margin 重疊

<!--more-->

### 狀況一

```html
<head>
  <style>
    .box1 {
      width: 100px;
      height: 100px;
      margin: 30px;
      background: pink;
    }
    .box2 {
      width: 100px;
      height: 100px;
      margin: 50px;
      background: lightblue;
    }
  </style>
</head>
<body>
  <div class="box1"></div>
  <div sclass="box2"></div>
</body>
```

</br>

<div style="width:100px;height:100px;margin:30px;background:pink"></div>
<div style="width:100px;height:100px;margin:50px;background:lightblue"></div>

可以發現到兩個 div 雖然都有設 margin，但兩者間的間距僅有 50px，是因為當上下都有 margin 時，會取最大的那個值。

### 狀況二

```html

<head>
	<style>
		.box1{
			width: 100px;
			height: 100px;
			background: pink;
		}
		.box2{
			width: 50px;
			height: 50px;
			margin: 20px;
			background: lightblue;
		}
	</style>
</head>
<body>
	<div class="box1"></div>
	<div sclass="box2"></div>
</div>
</body>

```

可以發現在子標籤中加上 margin，會連帶父標籤受影響。解決方式是在父標籤加上 `overflow: hidden;` 的屬性。
</br>

<div style="width:100px;height:100px;background:pink;overflow: hidden;">
    <div style="width:50px;height:50px;background:lightblue;margin:20px"></div>
</div>

---

### 當要顯示的列表不想超出既定範圍時(新聞、消息列表)

一般什麼樣式都沒指定的狀況下：

```html
<head>
  <style>
    ul {
      width: 200px;
      height: 100px;
      background: salmon;
    }
  </style>
</head>
<body>
  <ul>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
  </ul>
</body>
```

</br>

<ul style="width:200px;height:100px;background:salmon;">
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
</ul>
<br />
<br />
<br />
<br />
<br />
<br />

將 li 也設置寬度，利用 `white-space: nowrap` 讓文字不換行、`overflow: hidden;` 將超出框的部分隱藏、`text-overflow: ellipsis;` 將超出框的部分用 `...` 取代：

```html
<head>
  <style>
    ul {
      width: 200px;
      height: 100px;
      background: salmon;
    }
    ul li {
      width: 200px;
      hite-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  </style>
</head>
<body>
  <ul>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
  </ul>
</body>
```

</br>

<ul style="width:200px;height:100px;background:salmon;">
    <li style="width:200px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li style="width:200px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li style="width:200px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">這是一個超出範圍的很長很長很長很長很長的範例文字</li>
</ul>

另外用了 `overflow: hidden;` 會影響 list-style，即當 ul 中的 li 的 overflow 為 hidden 的時候， list-style 不起作用，不會顯示前面的點、圈等樣式。解決辦法是在 ul 或 li 內加入屬性 `list-style-position: inside;` 即可。

```html
<head>
  <style>
    ul {
      width: 200px;
      height: 100px;
      background: salmon;
    }
    ul li {
      width: 200px;
      hite-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      list-style-position: inside;
    }
  </style>
</head>
<body>
  <ul>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li>這是一個超出範圍的很長很長很長很長很長的範例文字</li>
  </ul>
</body>
```

</br>

<ul style="width:200px;height:100px;background:salmon;">
    <li style="width:200px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; list-style-position:inside;">這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li style="width:200px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; list-style-position:inside;">這是一個超出範圍的很長很長很長很長很長的範例文字</li>
    <li style="width:200px; white-space:nowrap; overflow:hidden; text-overflow:ellipsis; list-style-position:inside;">這是一個超出範圍的很長很長很長很長很長的範例文字</li>
</ul>
