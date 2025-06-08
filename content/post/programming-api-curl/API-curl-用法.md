---
title: "[API] curl 使用指南"
categories: ["Programming"]
date: 2020-07-02 23:00:00
slug: "api-curl"
---

## curl

curl 可以用來請求 Web 服務器、支持文件的上傳和下載。它的名字就是 client + URL 的意思。

<!--more-->

### 基本指令格式

```bash
$ curl [options] [URL]
```

### HTTP request

```bash
-X/--request [GET|POST|PUT|DELETE|…]		使用指定的 http method 發出 http request

-H/--header		設定request 裡的 header

-i/--include		顯示response 的 header

-d/--data		設定 http parameters

-v/--verbose		輸出比較多的訊息

-u/--user		使用者帳號、密碼

-b/--cookie		cookie
```

#### GET

```
$ curl https://github.com/ulahsieh
```

不帶有任何參數時，curl 就是發出 GET 請求，可以方便你測試 http server 是否運作正常。

#### POST

Http 參數可以直接加在 url 的 query string，也可以用 `-d` 帶入參數間用 `&` 串接，或使用多個 `-d`

```bash
curl -X POST -d "param1=value1&param2=value2"
```

- 檔案上傳

```bash
$ curl -X POST -F 'file=@./upload.txt' http://www.example.com/upload.php
```

-F 是使用 http query parameter 的方式，指定檔案位置的參數要加上 @

#### 認證

許多服務，需先進行登入或認證後，才能存取其 API 服務。可以透過 cookie、session 或在 header 加入 session key、認證的 token 來驗證。

- Session  
  如果是用 session 記錄使用者登入資訊，server 會傳一個 session id 給前端，前端需要在每次跟後端的 requests header 中置入此 session id，後端便會以此 session id 識別前端。

```bash
curl --request GET 'http://www.example.com/api/users' --header 'sessionid:xxxxxxx'
```

- Cookie  
  在認證後，後端會回傳一個 cookie，把該 cookie 存成檔案，當要存取需要任務的 url 時，再用 -b cookie_file 的方式在 request 中植入 cookie 即可正常使用。

```bash
# 將cookie存檔
curl -i -X POST -d username=ula -d password=1234 -c  ~/cookie.txt  http://www.example.com/auth
# 載入cookie到request中
curl -i --header "Accept:application/json" -X GET -b ~/cookie.txt http://www.example.com/users/1
```

### 其他常用到的參數

```
--user-agent <string>	設置用戶代理發送給服務器

--header <line>	自定義頭信息傳遞給服務器

-I	只顯示 response header

-u/--user <user[:password]>	設置服務器的用戶和密碼

-x <IP>	表示使用這個代理IP去請求其他的網頁

-s	靜默模式，不顯示返回的一大堆頁面內容

-o <檔案名稱>	取得網頁內容，輸出至檔案

-L	表示如果在response header中如果有 location 的話就直接轉向到 location的地址 (redirect地址)
```

#### 憑證錯誤的問題

如果有遇到以下錯誤，代表 curl 不認得 CA 憑證：
![](https://imgur.com/JhmHkDA.png)
要避免這個情況的話，需要在 curl 指令後面加上 `-k` 或 `–insecure` 參數，這樣 curl 便不會檢查 SSL 的有效性，例如：

`$ curl -k https://github.com/ulahsieh`
`$ curl --insecure https://github.com/ulahsieh`

![](https://imgur.com/jlg54zn.png)

## Reference

- [testing-rest-with-curl-command](http://blog.kent-chiu.com/2013/08/14/testing-rest-with-curl-command.html#header)
