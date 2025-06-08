---
title: "[CSS] 移入轉換文字內容"
categories:
  - Programming
tags: ["CSS"]
date: 2020-10-16 16:11:00
slug: "css-display-font-when-hover"
---

移入轉換其實很簡單，不需要算什麼正負位移，只要利用 display: none 屬性就可以輕鬆達成。先來看完成的範例：

<!--more-->

<head>
<style>
.nav2{
    width: 600px;
    height: 20px;
    margin: 0px auto;
    border-bottom: 8px red solid;
    padding-left: 10px;
}
.nav2 a{
    height: 20px;
    width: 80px;
    float: left;
    margin-right: 1px;
    text-decoration: none;
    text-align: center;
    background: rgb(223,223,223);
}
.nav2 a .en{
    display: block;
    color: black;
    font-size: 10px;
    line-height: 20px;
}
.nav2 a .cn{
    display: none;
    color: black;
    font-size: 10px;
    line-height: 20px;
}
.nav2 a:hover{
    background: red;
}
.nav2 a:hover .en{
    display: none;
    color: white;
}
.nav2 a:hover .cn{
    display: block;
    color:white;
}
</style>
</head>
<div class="nav2">
    <a href="#"><span class="en">Home</span><span class="cn">首頁</span></a>
    <a href="#"><span class="en">About</span><span class="cn">關於</a>
    <a href="#"><span class="en">Services</span><span class="cn">服務</a>
    <a href="#"><span class="en">Clients</span><span class="cn">客戶</a>
    <a href="#"><span class="en">Contact</span><span class="cn">聯絡</span></a>
</div>
      
<br>

代碼如下：

```html
<!DOCTYPE HTML>
<html>
    <head>
        <meta charset="utf-8"/>
        <style>
            .nav2{
                width: 916px;
                height: 20px;
                margin: 0px auto;
                border-bottom: 8px red solid;
                padding-left: 10px;
            }

            .nav2 a{
                height: 20px;
                width: 80px;
                float: left;
                margin-right: 1px;
                text-decoration: none;
                text-align: center;
                background: rgb(223,223,223);

            }
            .nav2 a .en{
                display: block;
                color: black;
                font-size: 10px;
                line-height: 20px;
            }

            .nav2 a .cn{
                display: none;
                color: black;
                font-size: 10px;
                line-height: 20px;
            }
            .nav2 a:hover{
                background: red;
            }
			<!-- 在游標移入時 .en 的樣式 -->
            .nav2 a:hover .en{
                display: none;
                color: white;
            }
			<!-- 在游標移入時 .cn 的樣式 -->
            .nav2 a:hover .cn{
                display: block;
                color:white;
            }
        </style>
    </head>
    <body>
        <div class="nav2">
            <!-- 利用兩個標籤轉換不同文字 -->
            <a href="#"><span class="en">Home</span><span class="cn">首頁</span></a>
            <a href="#"><span class="en">About</span><span class="cn">關於</a>
            <a href="#"><span class="en">Services</span><span class="cn">服務</a>
            <a href="#"><span class="en">Clients</span><span class="cn">客戶</a>
            <a href="#"><span class="en">Contact</span><span class="cn">聯絡</span></a>
        </div>
    </body>
<html>
```
