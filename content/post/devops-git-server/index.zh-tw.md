---
title: "[Git] Build a Git Server on Ubuntu"
tags:
  - Git
categories:
  - DevOps
date: 2020-03-14 19:48:00
slug: build-git-server-on-ubuntu
---

## Step 1: 電腦/VM 環境設置

- 作業系統選擇 Ubuntu 或其他 Linux 版本
- 開啟 SSH port
- 新增使用者帳號 git

<!--more-->

```
sudo adduser --home /home/git --disabled-password git
//--disabled-password 表示取消用密碼登入
```

</br>

{{< notice note >}}
small tips:
若開發團隊僅有兩三人，那不建立 git 帳號而是建立使用者個別的帳號也行。但考慮到日後可能擴編或是人員交替等等的因素，可能會增加管理的負擔。
{{< /notice >}}

## Step 2: 安裝所需套件

- 安裝 git

```
sudo apt-get install git
```

- 安裝 Gitosis

```
sudo apt-get install python-setuptools
cd /tmp
git clone https://github.com/tv42/gitosis.git
cd gittosis
sudo python setup.py install
```

承第一步結尾，為了使用及管理上的方便，Gitosis 便應運而生，Gitosis 是一套用來管理授權金鑰文件和實現簡單連接限制的腳本。可以輕鬆的管理每個使用者，而不需實際在主機上新增/移除帳號，這樣 user 皆可用 git@server:ServerIP.git 這樣的格式來取得或上傳版本庫到 Server 上。

- 在使用者家目錄 `/home/git/` 下新增資料夾 repositories 當作版本庫位置

Gitosis 預設會把 `/home/git` 這個目錄裡面的 repositories 資料夾當成所有 repository 的根目錄，以後所有的專案就都放在裡面。

## Step 3: git client 端的動作

每個要使用 git server 的使用者，將會擁護一把自己的金鑰，若同一使用者使用多台電腦，則需將同一把的私鑰複製到同路徑下便可連線。

- 開啟 git bash 產生一對金鑰

```
ssh-keygen -t rsa -C "xxx@email.com"
// -C 參數用於指定這個金鑰的識別碼，預設是「帳號@主機名稱」，但為了好管理，所以建議用使用者的唯一識別值如 email
```

- 輸入金鑰存放位置
  若沒輸入則存在預設位置

![](https://imgur.com/0ppmntS.png)

- 輸入密碼
  這裡不需設定，直接按兩次 enter 即可，這樣日後在 push 上 server 時，就不用輸入密碼了

![](https://imgur.com/thA3DK7.png)

- 產生金鑰
  產生完成後會有兩個檔案，副檔名 .pub 為公鑰，沒有副檔名的是私鑰，公鑰交給(寄給) git server 管理者，私鑰則好好保存在欲使用的電腦中。

![](https://imgur.com/28JswVa.png)

## Step 4: 管理者將使用者們的公鑰傳至 git server

- 使用 pscp 將檔案傳至 git server

```
pscp user1.pub git@ServerIP:/tmp/user1.pub
//將 user1.pub 以 git 帳號丟到 server 上的 /tmp 目錄下，若需重新命名，則直接在 /tmp/xxx.pub 中打上欲命名的檔名
```

- 輸入密碼以開始上傳

![](https://imgur.com/3GywqeJ.png)

## Step 5: 完成 Gitosis 的設定

Gitosis 有個很酷的管理方式，因為它本身就是一個 Git Repository，只要 clone 到管理者的電腦，修改裡面的檔案，再 push 回 Git Server，就會能完成如新增使用者公鑰、更改專案權限等設定。

- 在管理者地端的電腦中 git clone Gitosis

```
cd /c/Projects
git clone git@ServerIP:gitosis-admin.git
```

- git clone 後可看到目錄下的資料結構

![](https://imgur.com/NFLalCV.png)

- 新增使用者
  keydir 目錄裡面存放目前所有 Git 使用者的公鑰，把新使用者的公鑰丟到 keydir 目錄下

![](https://imgur.com/4GM2rl3.png)

- 編輯 gitosis.conf
  gitosis.conf 就是主要的設定檔，我們可以編輯裡面的設定來加入專案或使用者。
  這是剛初始化好的內容，裡面指定了 gitosis-admin 管理員的公鑰名稱(只允許一個管理員來管理 gitosis-admin)

![](https://imgur.com/NkLcrPE.png)

編輯 gitosis.conf，加入專案名稱和擁有寫入權限的使用者；
　　 1. [群組名稱] 可自定  
　　 2. members 為成員的公鑰識別碼 (建立金鑰時的 -C 參數值) 多個請用空白分隔  
　　 3. writable 為專案名稱

![](https://imgur.com/EYbI7GT.png)

- 提交更新並同步回 git server

```
git add
git commit -m "add an user and create a develop group"
git push origin master
```

到這個步驟，git server 就算是建完了!

## Step 6: 使用者使用

- push (在專案下)

```
git remote add origin git@ServerIP:project1.git
git push origin master
```

- pull

```
git clone git@ServerIP:project1.git
```
