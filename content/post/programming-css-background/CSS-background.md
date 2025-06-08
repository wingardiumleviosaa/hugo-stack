---
title: "[CSS] background"
categories:
  - Programming
tags: ["CSS"]
date: 2020-07-20 17:00:00
slug: "css-background"
---

<!--more-->

## background

- 指定顏色

```html
<!--html-->
<div id="box1">box1</div>
<div id="box2">box2</div>
<div id="box3">box3</div>
<div id="box4">box4</div>
```

</br>

```css
/* style.css */
#box1 {
  background: yellow;
}
#box2 {
  background: #2ab4b4;
}
#box3 {
  background: rgb(117, 107, 255);
}

#box4 {
  background: rgba(117, 107, 255, 0.3); /* a 指透明度，值從 0 到 1*/
}
```

![](https://imgur.com/JqH1PAs.png)

- 指定背景圖片

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg");
}
```

![](https://imgur.com/JJeq55s.png)

## background-repeat

背景平舖方式，默認為 repeat，可以接受的屬性值有`no-repeat/ repeat-x/ repeat-y`，其中 repeat-x 為水平方向重複，repeat-y 則為垂直方向重複。

![](https://imgur.com/sEPpMjb.png)

## background-position

可能的屬性值有 `center center / top center / bottom center`，第一個值為水平方向、第二個值為垂直方向。

1. center center 指定圖片置中。

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg");
  background-repeat: no-repeat;
  background-position: center center;
  height: 500px;
}
```

![](https://imgur.com/vggM3P1.png)

2. top center 指定圖片靠上置中對齊

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg");
  background-repeat: no-repeat;
  background-position: top center;
  height: 500px;
}
```

![](https://imgur.com/HrZij8s.png)

3. bottom center 指定圖片靠下置中對齊

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg");
  background-repeat: no-repeat;
  background-position: bottom center;
  height: 500px;
}
```

![](https://imgur.com/fWHk3u4.png)

## 簡寫

`backgound-repeat` 以及 `backgound-position` 可以直接表示在 `background` 屬性中，例如：

```css
background: url("./lemon.jpg") no-repeat center center;
```

這三個屬性值沒有特性順序，也可以將 `no-repeat` 寫在 `url("./lemon.jpg")` 前。

## background-size

- 用 pixel 表示

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg") no-repeat center center;
  background-size: 300px 100px
  height: 500px;
}
#box2 {
  background: #2ab4b4;
}
```

![](https://imgur.com/trKw6UL.png)

- 用百分比表示

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg") no-repeat center center;
  background-size: 50% 80%
  height: 500px;
}
#box2 {
  background: #2ab4b4;
}
```

![](https://imgur.com/UH2ANRV.png)

- contain  
  縮放圖片以完全裝入背景，背景區可能會留白。

```css
/* style.css */
#box1 {
  background: url("./lemon.jpg") no-repeat center center;
  background-size: contain;
  height: 200px;
}
#box2 {
  background: #2ab4b4;
}
```

![](https://imgur.com/NXi5EEu.png)

- cover  
  縮放圖片以完全覆蓋背景區，圖片可能部分會被裁切。
