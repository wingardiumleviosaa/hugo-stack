---
title: "[Node.js] npm 簡介"
tags:
  - Node.js
categories:
  - Programming
date: 2020-06-25 16:44:00
slug: node-npm-intro
---

## npm

Node Package Manager，package 就是別人寫好的套件，npm 就是幫你管理 Node.js 套件的管理系統。裝完 Node.js 就會自動把 npm 裝好，下 `npm -v` 可以查看版本。

<!--more-->

### npm init

第一次使用 npm 套件的話下 `npm init` 初始化，填寫專案相關資訊後，就會自動產生 `package.json` 的檔案。

<!--more-->

### package.json

如果在程式中使用很多套件，專案就會變得非常的大，在搬遷或是想 git push 分享的話會變得很麻煩。此時只需要一個用來紀錄專案中使用的套件列表以其版本，就能解決。
當別人拿到你的專案的時候，只要下 `$ npm install` 就會自動安裝完所有所需套件了！

![](https://imgur.com/0d5ApKr.png)

- dependencies：紀錄專案使用了哪些套件。
- devDependencies：紀錄**開發中**用的套件。
- script：
  可以在下各種 npm 　指令時依照 script 做一些事情。例如說一個程式的進入點除了可以在專案描述的時候特別指出 main．另外就是在 script 寫自動執行入口程式的腳本。

![](https://imgur.com/NtBC8vZ.png)

之後只要下 `$ npm run start` 就能跑起程式了，`npm run start` 算是約定俗成的入口程式指令。

### npm install

接著只要在專案目錄下，下 `npm install <moduleName>` 安裝套件。裝完後目錄下會多了一個 `package-lock.json` & `node_modules/`。

- **package-lock.json** 裡記錄了安裝的套件中各自又依賴了哪些套件。
- **node_modules/** 就是存放各套件程式碼的資料夾，通常會很大、不會被推到 git server，要 git pull 下來使用的人只需用 package.json 透過 npm install 來安裝指定套件。
  p.s. 如果是在子資料夾下 npm install，則會去記錄在上一層專案目錄下的 package.json 中，如果堅持要把 package 裝在子目錄下的話，就在子目錄下下 npm init 即可。

### --save 自動幫你加進 package.json

開發的途中只要隨時有新的套件想安裝的話，不需要手動去更改 package.json 的檔案，只需要在 `npm install` 的時候加上 `--save` 的參數即可。

```
$ npm install http-server --save
$ npm install left-pad --save-dev //自動記錄在 devDependencies 區域．僅供開發時用到的 module
```

</br>

{{< notice warning >}}
現在新版的 npm 不用加 --save 參數也會自動 save 到 package.json 了。
{{< /notice >}}

---

## yarn：除了 npm 的另一種選擇

**yarn 比 npm 還快**，是一個新出來、由 facebook 開發的開源套件管理。

1. 裝完 yarn 後，下 `$ yarn -v` 可以查看版本。
2. `$ yarn init` 初始化專案
3. 使用 `$ yarn add <module name>` 安裝套件，且會自動在 package.json 檔案中更新
4. 直接下 `$ yarn` 就能根據 package.json 裝完所需套件

## Source

[All] 此篇為觀看 Lidemy JS102 的筆記，圖片來源取自上課影片
