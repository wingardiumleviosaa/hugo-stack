---
title: '[Hexo] 設定可切換的 Dark Mode'
categories: ["Blog"]
tags: ["Hexo"]
date: 2021-10-31 20:50:00
slug: hexo-darkmode
---
在 NexT 主題中引入 Darkmode.js 以支持網頁的 Dark Mode。
<!--more-->
打開 `.\themes\next\layout\_scripts` 文件夾內的 `vendors.njk` 文件，在末尾添加以下代碼：

```
<script src="https://cdn.jsdelivr.net/npm/darkmode-js@1.5.7/lib/darkmode-js.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/darkmode-js@1.5.7/lib/darkmode-js.min.js"></script>
<script>
  function addDarkmodeWidget() {
    const options = {
      bottom: '64px', // default: '32px'
      right: 'unset', // default: '32px'
      left: '32px', // default: 'unset'
      time: '1s', // default: '0.3s'
      mixColor: '#fff', // default: '#fff'
      backgroundColor: '#fff',  // default: '#fff'
      buttonColorDark: '#100f2c',  // default: '#100f2c'
      buttonColorLight: '#fff', // default: '#fff'
      saveInCookies: true, // default: true,
      label: '🌓', // default: ''
      autoMatchOsTheme: true // default: true
    }
    const darkmode = new Darkmode(options);
    darkmode.showWidget();
  }
  window.addEventListener('load', addDarkmodeWidget);
</script>
```

p.s. 用此方式設定時，此代碼和在 Next 中插入背景圖片以及透明化會有衝突，但剛好目前都沒有該設定，就直接套用了。