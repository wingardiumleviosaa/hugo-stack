---
title: "[CSS] 垂直置中問題"
categories:
  - Programming
tags: ["CSS"]
date: 2020-10-19 09:38:46
slug: "css-virtical-and-center-alignment"
---

一般在盒內文字想達成垂直置中的效果我們使用 `line-height` 將高度設置與父元素一樣即可

<!--more-->

<div style="width:50px; height:50px; background: lightpink; line-height: 50px; text-align:center;">hello</div>
<br>

```html
<div
  style="width:50px; height:50px; background: lightpink; line-height: 50px; text-align:center;"
>
  hello
</div>
```

但如果遇到多行文字、圖片或其他元素，我們就需要使用 padding 或是 margin 來調整；

<div style="width:100px; height:100px; background: lightpink;overflow:hidden;">
  <div style="width:40px; height:40px; background: lightblue; margin: 30px;"></div>
</div>
<br>
<div style="width:40px; height:40px; background: lightpink;padding:30px;">
  <div style="width:40px; height:40px; background: lightblue;"></div>
</div>

<br>

```html
<!-- margin -->
<div style="width:100px; height:100px; background: lightpink;overflow:hidden;">
  <div
    style="width:40px; height:40px; background: lightblue; margin: 30px;"
  ></div>
</div>
<!-- padding -->
<div style="width:40px; height:40px; background: lightpink;padding:30px;">
  <div style="width:40px; height:40px; background: lightblue;"></div>
</div>
```

不過此時會遇到的問題是 --- 當頁面中的父級元素改變高度時，其先前設的子元素垂直置中的效果就必須重設。

## 解決方式 - 使用 vertical-align

首先 vertical-align 的使用概念是針對兩個擁有 inline 或是 inline-block 的元素保持垂直置中，舉例如下:

<img src="https://imgur.com/EgdquWK.png" style="height: 150px;">

```html
<img src="image.jpg" style="height:150px; vertical-align:middle;" /><span
  style="vertical-align:middle;"
  >I'm a fun guy.</span
>
```

我們會利用這個概念來解決上面問題

在欲垂直置中的子元素旁我們要建一個沒有寬度、高度與父元素同等的標籤，就能以此為對齊基準在父元素中垂直置中了。

![](https://imgur.com/0Xijpi0.png)

```html
<head>
  <style>
    div {
      width: 400px;
      height: 300px;
      background: lightyellow;
      text-align: center;
    }
    div span {
      display: inline-block;
      vertical-align: middle;
      height: 100%;
    }
    div img {
      vertical-align: middle;
      height: 100px;
    }
  </style>
</head>
<body>
  <div>
    <span></span>
    <img src="image.jpg" />
  </div>
</body>
```

## 常見範例

排列式商品頁面中的商品圖片不見得一致，此時這個方法就派上用場了，範例以及代碼如下；

![](https://imgur.com/wTXfaf9.png)

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="description" content="test" />
    <title>test</title>
    <style>
      * {
        margin: 0px;
        padding: 0px;
      }

      .box {
        width: 608px;
        height: 506px;
        margin: 10px auto;
      }

      .box dl {
        float: left;
        border: 2px grey dotted;
        width: 200px;
        height: 250px;
        margin: 0px -2px -2px 0;
      }

      .box dl dt {
        height: 180px;
        text-align: center;
      }

      .box dl dt span {
        height: 100%;
        display: inline-block;
        vertical-align: middle;
      }

      .box dl dt a {
        display: inline-block;
        vertical-align: middle;
      }
      .box dl dd {
        text-align: center;
      }

      .item {
        font-size: 18px;
      }

      .price {
        font-size: 16px;
        color: darkred;
        font-family: Arial, Helvetica, sans-serif;
        margin-top: 10px;
      }
    </style>
  </head>
  <body>
    <div class="box">
      <dl>
        <dt>
          <span></span>
          <a href="#"><img src="bread1.PNG" /></a>
        </dt>
        <dd>
          <div class="item">溫莎麵包</div>
          <div class="price">NTD$ 50</div>
        </dd>
      </dl>
      <dl>
        <dt>
          <span></span>
          <a href="#"><img src="bread2.PNG" /></a>
        </dt>
        <dd>
          <div class="item">法國香頌</div>
          <div class="price">NTD$ 55</div>
        </dd>
      </dl>
      <dl>
        <dt>
          <span></span>
          <a href="#"><img src="bread3.PNG" /></a>
        </dt>
        <dd>
          <div class="item">大菠蘿</div>
          <div class="price">NTD$ 45</div>
        </dd>
      </dl>
      <dl>
        <dt>
          <span></span>
          <a href="#"><img src="bread4.PNG" /></a>
        </dt>
        <dd>
          <div class="item">巧克粒圈</div>
          <div class="price">NTD$ 45</div>
        </dd>
      </dl>
      <dl>
        <dt>
          <span></span>
          <a href="#"><img src="bread5.PNG" /></a>
        </dt>
        <dd>
          <div class="item">花生什麼事</div>
          <div class="price">NTD$ 40</div>
        </dd>
      </dl>
      <dl>
        <dt>
          <span></span>
          <a href="#"><img src="bread6.PNG" /></a>
        </dt>
        <dd>
          <div class="item">桂圓三兄弟</div>
          <div class="price">NTD$ 55</div>
        </dd>
      </dl>
    </div>
  </body>
</html>
```
