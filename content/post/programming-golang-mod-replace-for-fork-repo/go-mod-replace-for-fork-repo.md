---
title: "[golang]  go mod replace 解决 fork repo 匯入問題"
tags:
  - Golang
categories:
  - Programming
date: 2022-01-06 14:10:00
slug: go-mod-replace-fork-repo
---

開發程式時使用 github 上各大神開發的第三方套件，有時候有自己的額外需求需要進行改造，所以 fork 到自己的 github 修改後，再 import 到自己的專案中。但當在進行構建的時候，報錯如下：

```
go get: github.com/bbb/xxx@v1.0.2: parsing go.mod:
        module declares its path as: github.com/aaa/xxx
                but was required as: github.com/bbb/xxx
```

<!--more-->

## 解决方式

使用 replace 將新的 package 去替換另一個 package，他們可以是不同的 package，也可以是同一個 package 的不同版本。
基本語法：

```bash
$ go mod edit -replace=old[@v]=new@v
```

新的 package 後面的 version 不可省略，可以是 release 版本號或是 git 的提交號（commit-id）。 （edit 所有操作都需要版本 tag）

也可以直接编辑 go.mod 文件：

```go
module test

go 1.16

require (
	github.com/gin-contrib/cors v1.3.1
	github.com/gin-gonic/gin v1.7.7
	github.com/sirupsen/logrus v1.8.1
	github.com/spf13/viper v1.10.1
	github.com/tebeka/selenium v0.9.9
	gopkg.in/alecthomas/kingpin.v2 v2.2.6
)

replace github.com/tebeka/selenium => github.com/ulahsieh/selenium v0.9.10-0.20220105013444-c7d3f285d0e7
// 注意版本號是用空格隔開
```

完成後在程式下面跑 `go mod tidy` 獲取新的套件並取代舊的，就大功告成啦！

## Reference

- https://github.com/Bpazy/blog/issues/164
- https://www.cnblogs.com/sunsky303/p/12150575.html
