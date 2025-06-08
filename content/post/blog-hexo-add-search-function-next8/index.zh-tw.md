---
title: '[Hexo] Add Search Function under NexT8 Theme'
categories: ["Blog"]
tags: ["Hexo"]
date: 2022-02-09 19:21:00
slug: hexo-add-search-bar
---
文章愈來愈多啦，只用目錄跟標籤找有如大海撈針，所以來幫部落格添加本站搜索的功能！在這邊記錄一下。

<!--more-->

## npm 安裝套件
Hexo 沒有本地搜尋的功能，但 NexT 主題已經內建搜尋功能的選項，不過需額外安裝套件。建議直接安裝 NexT 釋出的 hexo-generator-searchdb 套件會比較穩定。
```bash
npm install hexo-generator-searchdb
```

## 修改 config 檔
可以直接在 `themes/_config.yml` 找到 search 的相關設置，將 `enable: false` 改成 `true`。
```yaml
# Local Search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: false
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```

## 重新啟動 hexo server
就簡簡單單的大功告成了，讚嘆 NexT!

![](https://imgur.com/Vr1z0AW.gif)

## Reference
- https://home.gamer.com.tw/artwork.php?sn=5273365