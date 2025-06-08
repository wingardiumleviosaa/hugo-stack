---
title: "[CSS] Margin & Border & Padding 比較"
categories:
  - Programming
tags: ["CSS"]
date: 2020-09-16 14:49:00
slug: "css-margin-border-padding"
---

## 比較

| padding | border | margin |
| ------- | ------ | ------ |
| 內邊距  | 邊距   | 外邊距 |

<!--more-->

<img src="https://imgur.com/ME7vlle.png" height=350px>

## padding

用於調整元素邊界與內容的間距。

### 單獨指定寫法

- padding-top: 30px;
- padding-right: 30px;
- padding-bottom: 30px;
- padding-left: 30px;

### 合併寫法

- padding: 30px; --> 上下左右皆為 30px
- padding: 30px 50px --> 上下 30px, 左右 50px
- padding: 30px 20px 10px --> 上 30px, 左右 20px , 下 10px
- padding: 10px 20px 30px 40px --> 分別對應為上右下左

## margin

用於調整元素之間的邊界間距。
屬性設定同 padding 有上右下左的單獨屬性值，或是合併簡寫寫法。

## border

border 是元素最外層的邊界。

### 單獨指定樣式寫法

- border-width: 5px 5px 5px 5px; --> 上右下左
- border-style: solid|dashed|double|dotted|hidden;
- border-color: black;
- border-top: 5px;
- border-right: 5px;
- border-bottom: 5px;
- border-left: 5px;

### 合併寫法

- border: 5px red solid; --> 邊框粗細 5px, 顏色 red, 樣式 solid;
