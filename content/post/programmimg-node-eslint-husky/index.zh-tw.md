---
title: "[Node.js] eslint & husky 簡介"
categories:
  - Programming
tags: ["Node.js"]
date: 2020-06-20 22:56:00
slug: "eslint-husky"
---

## eslint (npm module)

eslint 等於 es + lint，用來檢查 Javascript coding style 的工具。

- es = ECMAScript, Javascript 的標準
- lint = 一個語法檢查的工具
<!--more-->

## husky (npm module)

husky 讓我們能在 package.json 配置 git hook 腳本。

---

使用 npm install 完成套件安裝 eslint + husky 後，就會在每次 commit 前自動檢查程式碼是否符合規範，符合才能提交成功。通常用在團隊開發時，可以統一程式碼風格。

![](https://imgur.com/PVW8W9i.png)

不符合規範時會在 console 印出錯誤訊息，

![](https://imgur.com/J4Y7BCS.png)

`前面的數字代表第幾行：後面的數字代表該行的幾個字發生錯誤。`
通常都**從後面開始修**，因為有些錯誤是要加空行，從前面改的話會導致後面的行號會亂掉。如果真的亂掉的話，那只要在重新下 git commit 一次，讓 eslint 重新檢查。

如果想讓 eslint 忽略某些檢查的話，例如要忽略可以宣告沒有被使用的變數，只要在最前面加上註解：
`// eslint-disable no-unused-vars`

![](https://imgur.com/jp7ckfz.png)

## Source

[All] 此篇為觀看 Lidemy MTR04 的筆記，圖片來源取自上課影片
