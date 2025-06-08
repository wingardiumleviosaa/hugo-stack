---
title: '[Hexo] NexT 更改字體'
categories: ["Blog"]
tags: ["Hexo"]
date: 2019-12-23 15:56:00
slug: "hexo-font"
---

Blog最重要的組成為文章，所以第一篇就決定來記錄我怎麼改網站的文字字體。

使用的方法是載入Google Font API -  
[Google Font](https://fonts.google.com/) 是一個包含近千種開源字體的字型庫，使用者除了可以免費下載外，還能在網站上自動產生程式碼，將字體內嵌到自己的網站上使用。

<!--more-->

目前網站上僅提供兩種繁體中文，思源黑體以及思源宋體：

![goole-font](https://imgur.com/1FqzQ9b.png)

以下為步驟：

## **選定字型** 

進入官網選擇喜歡的字型，依照網站樣式需求可以選擇多個，如文章內文中文欲使用思源宋體、英文欲使用Lato...  
請在字體右上角點選  `+`  加入選擇清單(底部提示框)
![google-font2](https://imgur.com/1ZrVFvc.png)

在已選擇清單中，切換分頁至 `CUSTOMIZE` ，勾選font-weight(regular400, bold700...)以及下方的語言(Latin & Chinese Traditional)

![](https://imgur.com/epwyaFs.png)

切回 `EMBED` 分頁，並複製網站自動編好的HTML code

![](https://imgur.com/BXchlhN.png)

## **編輯本地端文件**

### _config.yml

綠底及紅底分別代表code的增減。需特別注意的是 `global` 的 `family` 參數建議輸入英文字體，因為中文字型庫通常會包含英文字，故如若設為中文字體，則網站全域的預設文字包含英文皆會使用指定的中文字體。  
另外下方還有 `title` 網站標題、 `headings` 標頭、 `posts` 文章、 `codes` 程式碼，可以特別指定字體。

```diff
# 文件位置：~/blog/themes/next/_config.yml

font:
- enable: false
+ enable: true

  # Uri of fonts host, e.g. //fonts.googleapis.com (Default).
  host:

  # Font options:
  # `external: true` will load this font family from `host` above.
  # `family: Times New Roman`. Without any quotes.
  # `size: x.x`. Use `em` as unit. Default: 1 (16px)

  # Global font settings used for all elements inside <body>.
  global:
    external: true
    family: Lato
    size:

  # Font settings for site title (.site-title).
  title:
    external: true
    family: Amatic SC
    size:

  # Font settings for headlines (<h1> to <h6>).
  headings:
    external: true
    family: Lato
    size:

  # Font settings for posts (.post-body).
  posts:
    external: true
    family:

  # Font settings for <code> and code blocks.
  codes:
    external: true
    family:

```

而中文字體則到下方文件設置：

### base.styl
```diff
# 文件位置: ~/blog/themes/next/source/css/_variables/base.styl

// Font families.

- $font-family-chinese      = "PingFang SC", "Microsoft YaHei";
+ $font-family-chinese      = "Noto Serif TC";
```


### head.swig

接著將步驟一Google Font所複製的 `EMBED` 代碼貼上到 `head.swig`

```yaml=1
# 文件位置: ~/blog/themes/next/layout/_partials/head/head.swig
...
<link href="https://fonts.googleapis.com/css?family=Noto+Serif+TC:400,500,700&display=swap&subset=chinese-traditional" rel="stylesheet">
```

最後重新佈署便完成了! :laughing: