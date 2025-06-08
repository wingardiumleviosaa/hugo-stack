---
title: "[CSS] padding & margin"
categories:
  - Programming
tags: ["CSS"]
date: 2020-07-20 17:07:00
slug: "css-padding-margin"
---

## Padding

設定定義元素邊框與元素內容間的距離。

<!--more-->

```css
/* style.css */
#box1 {
  background: palegreen;
  width: 100px;
  height: 100px;
  padding: 30px;　/* 或填百分比 */
}
```

可以發現 box1 元素的字距離邊框有 30px 的邊距

![](https://imgur.com/ZvShxcR.png)

### 其他寫法

```
padding: 30px;
```

- padding 四個邊都是 30px。

```
padding: 25px 50px 75px 100px;
```

- 上填充為 25px
- 右填充為 50px
- 下填充為 75px
- 左填充為 100px

```
padding:25px 50px 75px;
```

- 上填充為 25px
- 左右填充為 50px
- 下填充為 75px

```
padding:25px 50px;
```

- 上下填充為 25px
- 左右填充為 50px

另外有針對四個邊的獨立寫法：

- padding-top：上方的內距
- padding-right：右方的內距
- padding-bottom：下方的內距
- padding-left：左方的內距
  四個屬性可以單獨使用也可以混搭使用。

## Margin

設定元素邊框與外面的距離。

```css
/* style.css */
#box2 {
  background: cadetblue;
  width: 100px;
  height: 100px;
  margin: 20px;　/* 或填 auto 自動調整 或 百分比 */
}
```

![](https://imgur.com/d168nEq.png)

### 其他寫法

margin 的其他寫法與 padding 完全相同。

- margin-top：設置元素的上外邊距。
- margin-right：設置元素的右外邊距。
- margin-left：設置元素的左外邊距。
- margin-bottom：設置元素的下外邊距。
  針對單一的 `margin` 屬性也可以設 `一個到四個` 的值。

### 瀏覽器自動建的 margin

瀏覽器會自動幫我們內建 body 的 margin，如下圖所標示

![](https://imgur.com/MibFlen.png)

此時只要在 css 中加入 body 的 selector，並把 `margin` 設成零就可以把邊框消除了。

```css
/* style.css */
body {
  margin: 0px;
}
```

![](https://imgur.com/6S4pUl0.png)
