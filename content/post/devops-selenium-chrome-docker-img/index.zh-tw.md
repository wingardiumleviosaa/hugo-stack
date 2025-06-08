---
title: "[Docker] 包 Selenium Chrome Docker Image"
tags:
  - Docker
  - Golang
categories:
  - DevOps
date: 2022-01-24 17:27:00
slug: build-selenium-chrome-docker-image
---

紀錄一下把 golang 改寫的 selenium 爬蟲程式包成 docker image 遇到的問題。

<!--more-->

## 最終的 Dockerfile

Chrome 跟 chromedrive 竟然都可以透過 apk 安裝，方便又輕鬆 :+1:

```docker
FROM golang:1.15.3-alpine3.12 AS build
WORKDIR /
COPY . .
RUN CGO_ENABLED=0 go build -o /grabreflow -ldflags="-s -w"


FROM alpine:edge
WORKDIR /
COPY ./config /config
COPY ./view /view
COPY --from=build /grabreflow /grabreflow
RUN apk add --no-cache chromium chromium-chromedriver && apk add wqy-zenhei --update-cache --repository https://nl.alpinelinux.org/alpine/edge/testing
ENTRYPOINT [ "/grabreflow" ]
```

## 踩坑紀錄

### 1. 使用 selenium 官方 docker image standalone-chrome 時無法讀寫檔案

#### Dockerfile

```docker
FROM golang:1.15.3-alpine3.12 AS build
WORKDIR /
COPY . .
RUN CGO_ENABLED=0 go build -o /grabreflow -ldflags="-s -w"

FROM selenium/standalone-chrome
WORKDIR /
COPY ./config /config
COPY ./view /view
COPY --from=build /grabreflow /grabreflow
COPY ./chromedriver /chromedriver
ENTRYPOINT [ "/grabreflow" ]
```

</br>

```sh
docker build -t grabreflow:0.9 -f seDockerfile .
docker run -it -d --name grabreflow -p 4444:4444 -p 8080:8080 --shm-size="2g" grabreflow:0.9
```

#### Error Logs

錯在 service.go 192 行拿 image 要裁切時是空的

```
INFO[01-24T03:40:49.157785Z] <nil>


2022/01/24 03:40:49 [Recovery] 2022/01/24 - 03:40:49 panic recovered:
GET /api/convergence/grabreflow/TBCBB2039913 HTTP/1.1
Host: 10.13.1.105:8080
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.5
Connection: keep-alive
Dnt: 1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0


interface conversion: interface is nil, not interface { SubImage(image.Rectangle) image.Image }
/usr/local/go/src/runtime/iface.go:454 (0x40b57b)
/pkg/service/service.go:192 (0xb678e9)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/context.go:168 (0xa6209a)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/recovery.go:99 (0xa77088)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/context.go:168 (0xa6209a)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/gin.go:555 (0xa6d29d)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/gin.go:511 (0xa6c9aa)
/usr/local/go/src/net/http/server.go:2843 (0x7bdbc2)
/usr/local/go/src/net/http/server.go:1925 (0x7b92cc)
/usr/local/go/src/runtime/asm_amd64.s:1374 (0x46b3a0)
```

#### service.go

上方的 log 的第一行 log INFO 印出的 nil 訊息就是下方程式中的第八行印的

```go=
scrnshot, _ := wd.Screenshot()
ioutil.WriteFile("test"+strconv.Itoa(i+1)+".png", scrnshot, 0666)
modal, _ := wd.FindElement(selenium.ByClassName, "modal-body")
loc, _ := modal.Location()
sz, _ := modal.Size()
file, _ := os.Open("./test" + strconv.Itoa(i+1) + ".png")
defer file.Close()
log.Info(file)
img, _ := png.Decode(file)
// service.go 198 行
sub_image := img.(interface {
SubImage(r image.Rectangle) image.Image
}).SubImage(image.Rect(loc.X, loc.Y, loc.X+sz.Width, loc.Y+sz.Height))
```

#### solution

![](https://imgur.com/ZX0nXdr.png)

進入 container 後發現根目錄下的確沒有圖片檔；可以推算此錯誤應該是權限問題，因為容器執行的 user 是 seluser，而程式執行檔放在根目錄下，所以指定的檔案也是在根目錄下做讀寫，因此會有權限問題。又另外看了一下這包 image 使用的基底作業系統竟然是這麼大的 Ubuntu 20，所以就決定不 debug，直接放棄這一包 image。
如果之後有要用這個映像檔的話，建議把執行檔放在 seluser 的家目錄下執行，或是在 Dockfile 的 entrypoint 前指定切換使用者為 root。

### 2. 使用自己包的 alpine edge 會遇到 chromedriver 叫不起來的錯誤

#### Error log

![](https://imgur.com/kBITplX.png)

```
Starting ChromeDriver 97.0.4692.99 (d740da257583289dbebd2eb37e8668928fac5ead-refs/branch-heads/4692@{#1461}) on port 9515
Only local connections are allowed.
Please see https://chromedriver.chromium.org/security-considerations for suggestions on keeping ChromeDriver safe.
[1643003263.594][SEVERE]: bind() failed: Address not available (99)
[1643003263.594][INFO]: listen on IPv6 failed with error ERR_ADDRESS_INVALID
[1643003263.594][SEVERE]: bind() failed: Address in use (98)
[1643003263.594][INFO]: listen on IPv4 failed with error ERR_ADDRESS_IN_USE
IPv4 port not available. Exiting...
```

然後回頭看網頁呼叫 api 有在持續 request 的感覺，等了大概半世紀，回頭看網頁停止運轉後，才丟出中間在爬的 log，最後停在 runtime error

```
[1643012436.436][DEBUG]: DevTools WebSocket Response: Runtime.evaluate (id=353) 65C32A9DA457F1E66E37A219E01492B4 {
   "result": {
      "description": "1",
      "type": "number",
      "value": 1
   }
}
[1643012436.436][INFO]: Done waiting for pending navigations. Status: ok
[1643012436.436][INFO]: [d57c16acae2654b7717bc67f35b0e974] RESPONSE ExecuteScript null
[1643012438.437][INFO]: [d57c16acae2654b7717bc67f35b0e974] COMMAND Quit {

}
[1643012438.488][INFO]: [d57c16acae2654b7717bc67f35b0e974] RESPONSE Quit
[1643012438.488][DEBUG]: Log type 'driver' lost 0 entries on destruction
[1643012438.488][DEBUG]: Log type 'browser' lost 35 entries on destruction


2022/01/24 08:20:38 [Recovery] 2022/01/24 - 08:20:38 panic recovered:
GET /api/convergence/grabreflow/TBCBB2039913 HTTP/1.1
Host: 10.13.1.105:8080
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.5
Cache-Control: no-cache
Connection: keep-alive
Dnt: 1
Pragma: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0


runtime error: invalid memory address or nil pointer dereference
/usr/local/go/src/runtime/panic.go:212 (0x44b932)
/usr/local/go/src/runtime/signal_unix.go:720 (0x44b7b2)
/go/pkg/mod/github.com/ulahsieh/selenium@v0.9.10-0.20220105013444-c7d3f285d0e7/service.go:275 (0xb63969)
/pkg/service/service.go:212 (0xb67b72)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/context.go:168 (0xa6209a)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/recovery.go:99 (0xa77088)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/context.go:168 (0xa6209a)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/gin.go:555 (0xa6d29d)
/go/pkg/mod/github.com/gin-gonic/gin@v1.7.7/gin.go:511 (0xa6c9aa)
/usr/local/go/src/net/http/server.go:2843 (0x7bdbc2)
/usr/local/go/src/net/http/server.go:1925 (0x7b92cc)
/usr/local/go/src/runtime/asm_amd64.s:1374 (0x46b3a0)

```

而 service.go 在 212 行的程式如下

```go
service.Stop()  // selenium service
```

推估是 selenium service timeout，但不知道為什麼爬蟲是在 timeout 前開始才動作的？無腦不想追究原因 XD

#### solution

爬文之後發現在 chrome argument 加上參數 `--allowd-ips` 或是 `--verbose` 可以解決，兩個參數都個別測試過可行，可以隨意挑一個加即可。

##### `--allowd-ips`

加上該參數，錯誤訊息依舊會出現，但是能成功爬取

```
Starting ChromeDriver 97.0.4692.99 (d740da257583289dbebd2eb37e8668928fac5ead-refs/branch-heads/4692@{#1461}) on port 9515
Only local connections are allowed.
Please see https://chromedriver.chromium.org/security-considerations for suggestions on keeping ChromeDriver safe.
[1643014821.860][SEVERE]: bind() failed: Address not available (99)
[1643014821.860][INFO]: listen on IPv6 failed with error ERR_ADDRESS_INVALID
ChromeDriver was started successfully.
```

##### `--verbose`

加上該參數，錯誤訊息依舊會出現，但是能成功爬取

```
Starting ChromeDriver 97.0.4692.99 (d740da257583289dbebd2eb37e8668928fac5ead-refs/branch-heads/4692@{#1461}) on port 9515
Only local connections are allowed.
Please see https://chromedriver.chromium.org/security-considerations for suggestions on keeping ChromeDriver safe.
[1643015241.625][SEVERE]: bind() failed: Address not available (99)
[1643015241.625][INFO]: listen on IPv6 failed with error ERR_ADDRESS_INVALID
ChromeDriver was started successfully.

```

### 3. 網頁有中文字的地方變成亂碼

![](https://imgur.com/Kxe4bqI.png)

上面的問題解決後成功爬取圖片，但又發現圖片中有中文的地方變成亂碼，原因是沒有安裝中文套件。

#### solution

在 container 中安裝中文套件可解決，建議直接加在 Dockerfile 預設安裝

```
apk add wqy-zenhei --update-cache --repository https://nl.alpinelinux.org/alpine/edge/testing
```

</br>

![](https://imgur.com/V4iwGil.png)

## Reference

- https://stackoverflow.com/questions/55844788/how-to-fix-severe-bind-failed-cannot-assign-requested-address-99-while
- https://www.jianshu.com/p/65cd4b138ee8
