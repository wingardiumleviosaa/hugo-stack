---
title: "[CSS] transition + :hover 做一個有質感的 button"
categories:
  - Programming
tags: ["CSS"]
date: 2020-07-24 10:11:05
slug: "css-transition-and-hover"
---

## transition

加上漸變效果，可以放慢某一個元素轉換效果的時間，增加質感。語法如下：
`transition: [property] [duration] [timing-function] [delay];`

<!--more-->

- property：指定 css 屬性的名稱，all 代表全部，並非所有的 css 屬性都可以使用，可動畫屬性[清單](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties)。
- duration：transition 效果的持續時間。
- timing-function：指定效果的轉速曲線，可使用數值如下：

  - ease
    - linear
    - ease-in
    - ease-out
    - ease-in-out
    - step-start
    - step-end
    - steps()
  - cubic-bezier()
    可以透過開發人員工具查看並拖曳曲線

    ![](https://imgur.com/5MLqxUV.gif)

- delay：定義多久之後開始發生 transition。

---

以下是沒有加上 transition 的畫面

```css
#box1 {
  background: lightgrey;
  width: 200px;
  height: 100px;
}
#box1:hover {
  background: darkcyan;
  color: white;
}
```

<br>

<head>
  <style>
    #box1 {
      background: lightgrey;
      width: 200px;
      height: 100px;
    }
    #box1:hover {
      background: darkcyan;
      color: white
    }
  </style>
</head>
<body>
  <div id="box1">
    box1
  </div>
</body>

---

接著來看加上 transition 的變化：
`transition: all 1s;` 第一個屬性值 all 代表對所有屬性作用，第二個值代表欲轉換的秒數。

```css
#box1 {
  background: lightgrey;
  width: 200px;
  height: 100px;
  transition: all 1s;
}
#box1:hover {
  background: darkcyan;
  color: white;
}
```

<br>

<head>
  <style>
    #box2 {
      background: lightgrey;
      width: 200px;
      height: 100px;
      transition: all 1s;
    }
    #box2:hover {
      background: darkcyan;
      color: white
    }
  </style>
</head>
<body>
  <div id="box2">
    box1
  </div>
</body>

---

## 做一個有質感的 button

```css
<head>
  <style>
    #box1 {
      width: 200px;
      height: 100px;
      text-align: center;
      line-height: 100px;
      border-radius: 30px;
      transition: all 0.5s;
      color: black;
      border: 3px solid darkcyan;
    }
    #box1:hover {
      background: darkcyan;
      color: white;
      cursor: point;
    }
  </style>
</head>
<body>
  <div id="box1">
    box1
  </div>
</body>
```

<br>

<head>
  <style>
    #box3 {
      width: 200px;
      height: 100px;
      text-align: center;
      line-height: 100px;
      border-radius: 30px;
      transition: all 0.5s;
      color: black;
      border: 3px solid darkcyan;
    }
    #box3:hover {
      background: darkcyan;
      color: white;
      cursor: point;
    }
  </style>
</head>
<body>
  <div id="box3">
    box1
  </div>
</body>

## Reference

[1] https://lidemy.com/courses/390445
[2] https://eyesofkids.gitbooks.io/css3/content/contents/transitions.html
[3] https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function
