---
title: "[CSS] 將重複邊框重疊"
categories:
  - Programming
tags: ["CSS"]
date: 2020-10-21 09:38:46
slug: "css-overlapping-repeat-border"
---

當兩個 inline-block 元素或是 具有 float 屬性的元素放在同一行時，**左邊元素的右邊邊框與右邊元素的左邊邊框會貼在一起**，導致原本想設計成 4px 的邊框變成 8px，如下範例；

<!--more-->

<ul style="height: 96px; width: 136px; margin: 10px auto; padding:0px;">
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px pink solid; padding: 0px;"></li>
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px lightblue solid; padding: 0px;"></li>
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px pink solid; padding: 0px;"></li>
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px lightblue solid; padding: 0px;"></li>
</ul>

```html
<style>
  .outter {
    height: 200px;
    width: 180px;
    background: white;
    margin: 20px auto;
    border: 1px black solid;
  }

  .inner {
    width: 60px;
    height: 60px;
    float: left;
    list-style: none;
    border: 4px black solid;
  }
</style>
<body>
  <div class="outter">
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
  </div>
</body>
```

## 解決方法

設定 margin 屬性為邊框值的相反數，就可以把重疊的部分邊框隱藏起來（被覆蓋），讓邊框以單線的形式顯示，比如重複的地方是右邊跟下邊，那就把 margin-right & margin-bottom 位移 -4px。。

<ul style="height: 92px; width: 132px; margin: 10px auto; padding:0px;">
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px pink solid; padding: 0px; margin: 0px -4px -4px 0px;"></li>
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px lightblue solid; padding: 0px; margin: 0px -4px -4px 0px;"></li>
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px pink solid; padding: 0px; margin: 0px -4px -4px 0px;"></li>
	<li style="width: 60px; height: 40px; float: left; list-style: none; border: 4px lightblue solid; padding: 0px; margin: 0px -4px -4px 0px;"></li>
</ul>

```html
<style>
  .outter {
    height: 200px;
    width: 180px;
    background: white;
    margin: 20px auto;
    border: 1px black solid;
  }

  .inner {
    width: 60px;
    height: 60px;
    float: left;
    list-style: none;
    border: 4px black solid;
    /* 邊框處理 */
    margin: 0px -4px -4px 0px;
  }
</style>
<body>
  <div class="outter">
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
  </div>
</body>
```
