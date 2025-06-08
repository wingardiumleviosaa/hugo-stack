---
title: "[CSS] border & outline"
categories:
  - Programming
tags: ["CSS"]
date: 2020-07-20 17:05:00
slug: "css-border-outline"
---

<!--more-->

## border

```html
<div id="box1">box1</div>
```

</br>

```css
#box1 {
  background: yellow;
  border: 20px solid green;
  /* 框限大小  樣式  顏色*/
  height: 30px;
}
```

![](https://imgur.com/sWjrSzS.png)

邊框是往外面長的，例如上面的例子
能發現到整個選取的地方高度總共是 70px

### broder-style

border style 可以為外框限加上不同樣式，以下示範使用開發人員工具上面改樣式。

![](https://imgur.com/FciPkEY.gif)

### border-radius

可以為外框的角加上弧度。

![](https://imgur.com/dLXfqpa.gif)

### 用 border 畫圓形

可以在沒有設 border 參數下直接使用 border-radius，將值設成 **50px** 或是 **50%**，即可畫一個圓形。

![](https://imgur.com/pwHCycd.png)

### 用 border 畫三角形

```css
/* style.css */
#box1 {
  background: yellow;
  width: 100px;
  height: 100px;
  border-top: 30px solid salmon;
  border-bottom: 30px solid turquoise;
  border-right: 30px solid peachpuff;
  border-left: 30px solid pink;
}
```

可以單獨為四個邊設定 border，每個邊會變成梯形；

![](https://imgur.com/lCqP5pH.png)

當中間的元素長寬都設 **0px** 時，四個邊會變成三角形；

![](https://imgur.com/NfeVO3V.png)

把除了要留住的三角形以外的邊以及元素本身顏色都設為 _transparent_ 透明；

![](https://imgur.com/ZTpmNjH.png)

調整各邊的大小可以得道不同長邊的三角形。

![](https://imgur.com/ZjQn33V.png)

## Outline

Outline 跟 Border 都是邊框，不一樣的是 Outline 是外框，不會改變元件的位置，而 border 會。

![](https://imgur.com/vU6Mq2K.gif)

{{< notice info >}}
不過實際上比較常用 border!
{{< /notice >}}
