---
title: "[Golang] Render Image in HTML"
tags:
  - Golang
categories:
  - Programming
date: 2023-06-05 16:19:00
slug: golang-render-img-in-html
---

## TL;DR

紀錄怎麼在 golang 的 gin api server 回傳帶有圖片的 api。

<!--more-->

## 程式碼

```go
c.HTML(http.StatusOK, "img.html", gin.H{
		"images": images,
})
```

</br>

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Grab Reflow History</title>
  </head>
  <body>
    <div class="margin-body">
      <div class="row">
        {{range .images}}
        <div class="image">
          <img src="data:image/png;base64,{{.}}" />
        </div>
        {{end}}
      </div>
    </div>
  </body>
</html>
```

## Reference

- https://ithelp.ithome.com.tw/articles/10269897
- https://stackoverflow.com/questions/57020601/how-to-render-show-images-in-html-created-on-the-fly-with-golang-and-gin-gonic
