---
title: "[Docker] 建立 golang 中使用到 oracle instant client 的 image"
tags:
  - Docker
  - Oracle
  - Golang
categories:
  - DevOps
date: 2021-09-07 22:05:00
slug: build-golang-and-oracle-instant-client-docker-image
---

紀錄一下建立的過程中總共遇到兩個問題：

<!--more-->

## standard_init_linux.go:xxx: exec user process caused "no such file or directory"

之前在 build golang 的時候沒有用到 C library，所以在編譯的時候 CGO 都是關閉的。但目前要 build 的這隻程式有使用到 oracle 第三方套件 [`godror`](https://github.com/godror/godror) 需要用到 C library，所以在使用原生 alpine image 時，跑 container 起來會遇到 `standard_init_linux.go:228: exec user process caused "no such file or directory"` 的錯誤。

### 概念

#### 靜態編譯 & 動態編譯

靜態編譯指的是，在編譯可執行文件的時候，將可執行文件需要調用的對應庫都集成到可執行文件內部，使得可執行文件不需要其他任何依賴就能運行。

默認情況下，golang 的編譯是動態編譯，通過環境變量 CGO_ENABLED 控制，默認開啟，允許在 Go 代碼中調用 C 代碼。

#### Alpine 鏡像

Alpine 是眾多 Linux 發行版中的一員，和 CentOS、Ubuntu 一樣，只是一個發行版的名字，號稱小巧安全，有自己的包管理工具 apk，開發者可以使用 apk 在基於 alpine 的鏡像中添加需要的包或工具。

相比於其他 Docker 鏡像，它的容量非常小，僅僅只有 5 MB 左右（對比 Ubuntu 系列鏡像接近 200 MB）

### 錯誤原因

然而動態編譯完成後的二進制文件，放在同是使用動態庫的 Alpine 基礎鏡像中運行會報錯：`standard_init_linux.go:211: exec user process caused "no such file or directory"`

但是放在 ubuntu 基礎鏡像中可以運行。原因是因為兩者使用的 C library 不一樣，在製作 Alpine 的時候，是基於`musl libc` 和 busybox 構建的，與基於標準 C 執行庫 GNU C library (glibc) 上編譯出來的應用程序不兼容，導致動態依賴的二進制文件在運行時找不到依賴的文件。

### 解決方式

1. 使用動態編譯後運行在大基礎鏡像中，即包含動態調用的 C 庫的基礎鏡像
2. 手動安裝 C Library，使用動態編譯後運行在小基礎鏡像，將鏡像分為 `build` 和 `run` 兩個階段。

為了追求更小的 docker image，這裡使用第二個方法。

![](https://imgur.com/aODpLcK.png)

範例一：

```docker
//Dockerfile

# build stage
FROM golang:alpine as builder
WORKDIR /go/src
COPY httpserver.go .
RUN go build -o httpd ./httpserver.go

# run stage
From alpine:latest
WORKDIR /root/
COPY --from=builder /go/src/httpd .
RUN chmod +x /root/httpd

ENTRYPOINT ["/root/httpd"]
```

範例二：

```docker
FROM alpine:edge AS build
RUN apk update
RUN apk upgrade
RUN apk add --no-cache go gcc g++
WORKDIR /
COPY . .
RUN CGO_ENABLED=1 go build /

FROM alpine:edge
WORKDIR /
COPY --from=build /reverseapi /reverseapi
ENTRYPOINT [ "/reverseapi" ]
```

### 補充

#### Alpine 鏡像的限製或者前提條件

是否可以使用 Alpine，大部分項目均可無視 musl libc 和 gnu libc 的區別，但是如果有相關的依賴 gnu libc，則需要慎重考慮是否一定需要使用 Alpine 鏡像，因為 Alpine 鏡像的小巧正是建立在 busybox 和 musl libc 基礎之上的，雖然在 Alpine 上也可以安裝 gnu libc，但是這種對應給人一種強烈的違和感，而且後續碰到的各種問題的概率較大。所以整體來說，如果沒有一定要使用 gnu libc 的項目可以考慮使用 Alpine 鏡像，否者還是不要使用較為穩妥。

#### 版本支持

需要注意的是不同的 Alpine 的版本當前所支持的 gcc/g++的版本有所不同，如果不是從源碼編譯的安裝的情況下，需要注意此方面的限制，具體的查詢請參考[這裡](https://pkgs.alpinelinux.org)

## Reference

- [https://www.ardanlabs.com/blog/2020/02/docker-images-part2-details-specific-to-different-languages.html](https://www.ardanlabs.com/blog/2020/02/docker-images-part2-details-specific-to-different-languages.html)
- [https://promacanthus.netlify.app/experience/golang/01-编译的坑/](https://promacanthus.netlify.app/experience/golang/01-%E7%BC%96%E8%AF%91%E7%9A%84%E5%9D%91/)
- [https://tonybai.com/2017/12/21/the-concise-history-of-docker-image-building/](https://tonybai.com/2017/12/21/the-concise-history-of-docker-image-building/)

- 關於 alpine 鏡像使用 gcc & g++ 要注意的問題
  [https://blog.csdn.net/liumiaocn/article/details/100903476](https://blog.csdn.net/liumiaocn/article/details/100903476)

---

## ORA-00000: DPI-1047: Cannot locate a 64-bit Oracle Client library: "Error loading shared library libnsl.so.1: No such file or directory (needed by /instantclient_18_5/libclntsh.so)".

在 image 中裝完 Oracle 的 instantclinet 後，直接跑起來會遇到找不到參考的 library 的錯誤

![](https://imgur.com/NU8XcVr.png)

官網雖然有[參考文檔](https://docs.oracle.com/en/database/oracle/oracle-database/18/lnoci/instant-client.html#GUID-7D65474A-8790-4E81-B535-409010791C2F)，但是因為我用的 base image 是 alpine，有些功能跟 library 放的地方跟其他 Linux 如 ubuntu 或 centos 不一樣，所以決定安裝該有的 Library 後，再自己手動建 symbol link。

最後的 Dockerfile 長這樣：

```docker
FROM alpine:edge AS build
RUN apk update
RUN apk upgrade
RUN apk add --no-cache go gcc g++
WORKDIR /
COPY . .
RUN CGO_ENABLED=1 go build /

FROM alpine:edge
WORKDIR /
# COPY ./config /config
RUN wget https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-basic-linux.x64-18.5.0.0.0dbru.zip && \
    unzip instantclient-basic-linux.x64-18.5.0.0.0dbru.zip && \
    apk add --no-cache libaio libnsl libc6-compat gcc && ln -s /usr/lib/* /instantclient_18_5 && \
    cd /instantclient_18_5 && \
    ln -s libnsl.so.2 /usr/lib/libnsl.so.1 && \
    ln -s /lib/libc.so.6 /usr/lib/libresolv.so.2 && \
    ln -s /lib/libc.musl-x86_64.so.1 /usr/lib/ld-linux-x86-64.so.2
ENV LD_LIBRARY_PATH=/instantclient_18_5
COPY --from=build /reverseapi /reverseapi
ENTRYPOINT [ "/reverseapi" ]
```

</br>

{{< notice info >}}
在測試的時候是安裝完整的 instantclinet，但應該可以裝 lite 版本的，以減少不必要的 image 大小。
{{< /notice >}}

</br>

{{< notice info >}}
在 debug 的時候可以直接進去 container 裡面找錯誤訊息中找不到的 library 放哪，然後在自己手動建 link
{{< /notice >}}

## Reference

- [https://stackoverflow.com/questions/53263972/oracle-on-alpine-linux](https://stackoverflow.com/questions/53263972/oracle-on-alpine-linux)

- 19 版的可以參考下面大大分享的
  [https://github.com/Shrinidhikulkarni7/OracleClient_Alpine](https://github.com/Shrinidhikulkarni7/OracleClient_Alpine)

- 關於 alpine 中下 ldconfig 的錯誤
  [https://stackoverflow.com/questions/36990951/ldconfig-seems-no-functional-under-alpine-3-3](https://stackoverflow.com/questions/36990951/ldconfig-seems-no-functional-under-alpine-3-3)
