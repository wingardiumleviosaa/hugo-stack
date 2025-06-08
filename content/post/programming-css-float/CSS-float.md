---
title: "[CSS] float"
categories:
  - Programming
tags: ["CSS"]
date: 2020-09-11 17:01:00
slug: "css-float"
---

## HTML Normal Flow / inline & block

一般 HTML 的 normal flow 為由左至右及由上至下排列，一般取決於元素的 display 屬性是 `inline` 或是 `block`

- block 元素：區塊元素，在文檔中自己佔一行。如 `<div>`、`<p>`、`<ol><ul><li>`、`<dl><dt><dd>`、`<table><tr><td>`、`<form>`。
- inline 元素：行內元素或內聯元素，可以在一行顯示。如 `<span>`、`<a>`、`<i>`、`<b>`。
- inline-block 元素；行內塊元素。如 `<img>`、 `<input>`。
<!--more-->

---

## Float 浮動

因為 HTML flow 的特性四個元素會由上至下排序

```
<div class="box">1</div>
<div class="box">2</div>
<div class="box">3</div>
<div class="box">4</div>
<div class="box">5</div>
```

```
.box{
  width: 50px;
  height: 50px;
  background-color: orange;
  border: 1px solid;
  margin: 10px;
}
```

<style>
.box{
  width: 50px;
  height: 50px;
  background-color: PaleGreen;
  border: 1px solid;
  margin: 10px;
}   
</style>

<div class="box">1</div>
<div class="box">2</div>
<div class="box">3</div>
<div class="box">4</div>
<div class="box">5</div>

此時如果想要將 div box 由左至右排序，則可以使用 `float` 屬性。

```
.box{
  width: 50px;
  height: 50px;
  background-color: orange;
  border: 1px solid;
  margin: 10px;
  float: left
}
```

<style>
.box2{
  width: 50px;
  height: 50px;
  background-color: PaleGreen;
  border: 1px solid;
  margin: 10px;
  float: left
}   
</style>

<div class="box2">1</div>
<div class="box2">2</div>
<div class="box2">3</div>
<div class="box2">4</div>
<div class="box2">5</div>

<br/> <br/><br/><br/><br/>

---

## Float 應用情景

1. 為了實現文字環繞效果
2. 為了設計 DIV 區塊水平排列 (後來已漸漸被 inline-block 取代)

以下為文繞圖效果：

<style>
img{
    float: right
}
.out{
    border: 4px solid yellow;
    padding: 10px
}
.clear{
  clear: both;
}
</style>
<div class="out">
<img src="https://imgur.com/cpUaqBE.png" style="height:100px">
<div style="background:PapayaWhip;color:black">
    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor. Aenean massa. Cumsociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. In enim justo, rhoncus ut, imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis pretium. Integer tincidunt. Cras dapibus. 
</div>
</div>

## Float 可能發生的錯誤

以下面的例子可以發現外面的框限沒有包覆到圖片，是因為浮動元素 img 在框線 div 的前面，而後面的框線 div 元素沒有使用浮動時，就會往上跑、疊在前一個元素的下方。

<style>
img{
    float: right
}
.out{
    border: 4px solid yellow;
    padding: 10px
}
</style>
<div class="out">

<div style="background:PapayaWhip;color:black">
    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor. Aenean massa. Cumsociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. In enim justo, rhoncus ut, imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis pretium. Integer tincidunt. Cras dapibus. 
</div>
<img src="https://imgur.com/cpUaqBE.png" style="height:100px">

</div>

<br/> <br/><br/> <br/>

## 清除浮動 － clear

clear 屬性用於清除浮動，可能的值有 left、right、both、none 以及 inherit（IE 不支援），其中以 both 最常用。

後面的元素被浮動元素覆蓋，做法是在兩元素中另外新增一個 div 標籤，並賦予屬性 clear，其值為 both。

```html
<style>
  img {
    float: right;
  }
  .out {
    border: 4px solid yellow;
    padding: 10px;
  }
  .cleafix {
    clear: both;
  }
</style>
<div class="out">
  <div style="background:PapayaWhip;color:black">
    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo
    ligula eget dolor. Aenean massa. Cumsociis natoque penatibus et magnis dis
    parturient montes, nascetur ridiculus mus. In enim justo, rhoncus ut,
    imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis
    pretium. Integer tincidunt. Cras dapibus.
  </div>
  <img src="https://imgur.com/cpUaqBE.png" style="height:100px" />
  <div class="clearfix"></div>
</div>
```

<style>
img{
    float: right
}
.out{
    border: 4px solid yellow;
    padding: 10px;
}
.clearfix{
    clear: both;
}
</style>
<div class="out">
<div style="background:PapayaWhip;color:black">
    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor. Aenean massa. Cumsociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. In enim justo, rhoncus ut, imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis pretium. Integer tincidunt. Cras dapibus.
</div>
<img src="https://imgur.com/cpUaqBE.png" style="height:100px">
<div class="clearfix"></div>
</div>

---

## inline-block

上面 float 的應用場景提到，在傳統的設計 div 區塊水平排列的時候會採用 float 屬性，讓每個 div 區塊產生浮動的效果，但很容易造成這些區塊以外的區塊被覆蓋，所以還使用 clear 屬性，清除後面的浮動的效果。

現在使用 display:inline-block 可以做到 div 區塊水平排列，而且不需要額外設定 clear 也不會讓後面的元素被浮上來的區塊蓋住。

以下範例可以說明 block, inline 以及 inline-block 屬性的區別

<div style="background:PapayaWhip;color:black">
    Lorem ipsum dolor sit amet, consectetuer adipiscing elit. Aenean commodo ligula eget dolor. Aenean massa. Cum <span class="ex1 style">inline</span>sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. In enim justo, rhoncus ut, imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede mollis pretium.<span class="ex2 style">block</span> Integer tincidunt. Cras dapibus. Vivamus elementum semper nisi. Aenean commodo ligula eget dolor.Cum sociis natoque penatibus et magnis dis parturien montes,  Aenean massa. Cum <span class="ex3 style">inline-block</span>sociis natoque penatibus tet magnis dis parturient montes, nascetur ridiculus mus.Aenean vulputate eleifend tellus. Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
</div>

<style>
.style{
    border:1px solid black;
    padding:20px;
    margin:5px;
    background-color: yellow;
}
.ex1{
    display:inline;
} 

.ex2{
    display:block;
} 

.ex3{
    display:inline-block;
} 
</style>
<br/>

```css
.style {
  border: 1px solid black;
  padding: 20px;
  margin: 5px;
  background-color: yellow;
}
.forInline {
  display: inline;
}

.forBlock {
  display: block;
}

.forInline-block {
  display: inline-block;
}
```

- inline 元素雖有設定 padding 及 margin，但元素上下並不會把其他行推開，會使得其他行被蓋到。
- block 元素雖然可以設定寬高、padding 及 margin，但其屬性會占滿整行，無法並排。
- **inline-block 可同時擁有 block 可設定寬高的屬性，但其排版仍像 inline 屬性，不會向右占滿整行。**
