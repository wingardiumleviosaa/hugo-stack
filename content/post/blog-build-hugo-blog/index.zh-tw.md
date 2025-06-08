---
title: "架 Hugo 網站並部署到 Netlify"
date: 2020-08-11T21:30:55+08:00
tags: ["Hugo"]
categories: ["Blog"]
slug: "Build-Hugo-with-Netlify"
---

## 前言

第一個 blog 是用功能強大的 Hexo + Github Pages 做出來的，Hexo 基於 Node.js，有功能齊全的主題如 NexT、和豐富的外掛，僅需在配置檔修改參數就可以達到高客製化的需求，連部署到網頁上都僅需要幾個包裝好的 command，非常容易上手。

<!--more-->

不過隨著文章增加，能發現在生成靜態網頁的速度變慢了很多！

Hugo 由 Golang 編寫，標榜為世界上最快的靜態網站建構框架。缺點是沒有預設的主題樣式，現成的樣板選擇也沒有 Hexo 那麼多，配置檔幾乎是從頭編寫，傳統單純部署在 github pages 的方式也是不那麼簡易。

{{< notice note >}}
新手適合 Hexo 配置簡單，而要求速度的人則選擇 Hugo。
{{< /notice >}}

比較下來決定使用 Hugo。原本應該是要把 Hexo 部落格搬移到 Hugo，但發現兩者個 markdown 格式有些差異。因為懶得研究更多用法，這邊索性紀錄一些生活吧。

---

## 環境建置

### 安裝 Go

到 [golang](https://golang.org/dl/) 下載作業系統對應的程式。

### 下載 Hugo

到 [這裡](https://github.com/gohugoio/hugo/releases) 下載作業系統對應的程式。下載後做以下設定：

1. 電腦本機新增資料夾 c:/Hugo
2. 在該資料夾內新增 bin, Sites 兩資料夾。
3. 將官網下載的 hugo 解壓縮後放入 c:/Hugo/bin 資料夾內。
4. 設置環境變數，在 PATH 參數加入 `C:\Hugo\bin`
5. 重開機
6. 確認 Hugo 安裝成功，開啟 cmd 輸入 `hugo version`，若出現下面訊息則代表安裝成功。

![](https://imgur.com/mfLLKmI.png)

### 建立網站

開啟 cmd，切換到剛剛建立的目錄 c:\Hugo\Sites，建立一個名稱為 [example]的測試網站，輸入：

```
hugo new site example
```

如果出現 `Configurations!`
字樣，表示已經生成相關資料夾與檔案於 c:\Hugo\sites\example 下。

### 套用主題

用 `git submodule` 把喜歡的主題下載下來

```
$ cd example
$ git submodule add https://github.com/varkai/hugo-theme-zozo　theme/
```

因為 theme 本身就為一個 repo，而自己本身也會是一個 repo，此時如果要在 repo 裡面使用到其他 repo 的話就必須使用 git submodule！當使用 submodule 下載後，會多出 `.gitmodules` 檔案，用來紀錄掛載 submodule 的路徑跟 url。

在 `config.toml` 中更改 theme 屬性值為欲套用的樣板資料夾名稱。

### 新增文章

輸入 `hugo new posts/xxxx.md`，會在文件夾 `content/posts` 下形成 markdown 文件，打開進行編輯即可。
文章前的 metadata 如下：

```
---
title: "Build Hugo Blog"				# 文章標題
date: 2020-08-11T21:30:55+08:00				# 發表日期
draft: false						# 設置草稿
tags: ["Hugo"]						# 文章標籤
slug: "Build-Hugo-with-Netlify"				# URL 名稱
---
```

### 測試運行

使用 `hugo server` 並在瀏覽器中開啟 `localhost:1313` 之後就可以看到靜態生成的網站了。

![](https://imgur.com/iOVhZF7.png)

### 部屬至 github

在網站資料夾下初始化 repo

```
$ git init
```

前往 github 創建一個 repository，不需要建 README.md。

![](https://imgur.com/8qzVGma.png)

接下回到本地端 repo 並使用以下指令：

```
git add .
git commit -m "first commit"
git remote add origin git@yiru:/yirooo/hugo_blog.git
git push -u origin master
```

這邊說明一下，因為我的電腦中同時存在兩個 git 帳號，所以在設 remote repo 時是根據 .ssh 中的 config 檔的 host 設的。可以參考這邊[文章](https://ulahsieh.github.io/it-multi-account.html)

![](https://imgur.com/lf3Bqmi.png)

### 部屬到 Netlify

可用 Netlify 託管靜態網站，Netlify 支持編譯 Github 倉庫的代碼，把 Hugo 網站源代碼上傳至 Github 用 Git 管理，然後在 Netlify 上發佈網站，就可以實時部屬最新網站內容，而不用使用傳統而複雜的 Git Pages。

1. 用 Github 帳戶登錄 Netlify
2. 選擇 `New site from Git`
3. 選擇 Github ，然後關聯包含網站源代碼的倉庫
4. 設置
   - Branch to deploy：master
   - Build command：hugo
   - Publish directory：public
   - HUGO-VERSION：0.69.2
5. Deploying，默默等待一兩分鐘，Netlify 隨機分配一個子域名 `*.netlify.com`，可以隨意修改 `*` 的內容。
6. 回到本地端修改 `config.toml` 的 `BaseURL` 參數為上一步驟的 URL。
7. 大功告成，之後在本地端做修改後 git push 上 github repo，Netlify 就會自動去同步內容。
