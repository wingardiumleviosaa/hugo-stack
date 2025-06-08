---
title: "[Hexo] 升級至 5.x 與 NexT 主題 8.x"
categories: ["Blog"]
tags: ["Hexo"]
date: 2021-10-31 17:23:00
slug: hexo-and-next-upgrade
---

## Hexo 升級

Hexo 版本及系統插件可以透過 npm 實現，請按照下列步驟執行：

<!--more-->

```bash
# 全局升級 hexo-cli
npm install hexo-cli -g

# 檢察已安裝的插件(package.json)是否可升級
npm install -g npm-check
npm-check

# 升級系統中的插件
npm install -g npm-upgrade
npm-upgrade

# 更新全局插件
npm update -g
npm update --save

# 查看 hexo 版本，看是否有升級成功
hexo v
```

更新完成後我就直接下 `hexo g` 了，因為前後版的配置有些許差異，所以報錯不須緊張，只要根據錯誤訊息修改已 deprecated 的配置項。

```
# Deprecated
external_link: true|false
# New option
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''

# Deprecated
use_date_for_updated: true
# New option
updated_option: date
```

## NexT 升級

1. 用不同於原先版本的名字，下載新的 v8 倉庫

```bash
git clone https://github.com/next-theme/hexo-theme-next themes/next8
```

如此，你可以在不修改原有的 NexT 舊版目錄的同時使用 next8 目錄中的新版本主題。

2. 在 hexo 的主配置文件設置主題：

```yaml
theme: next8
```

如此，新主題將在生成站點時被加載。如果升級途中遇到了任何錯誤、或只是不喜歡這一新版本，可以隨時切換回舊版本。

3. 比對新舊主題的 \_config.yml，針對客製化的樣式進行設定，如字體、icon...

## Reference

- https://www.cylong.com/blog/2020/08/10/update-hexo-next/
