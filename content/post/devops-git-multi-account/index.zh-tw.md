---
title: "[Git] 在同一台電腦中設置多個 Git 帳號"
tags:
  - Git
categories:
  - DevOps
date: 2020-08-05 17:46:00
slug: git-multi-account-on-same-pc
---

## 為每個帳號產生 ssh key

```
ssh-keygen -t rsa -C "userName@address"
```

<!--more-->

## 產生完畢後，將公鑰放到對應帳號的 github 中

![](https://imgur.com/lNXpiXi.png)

## 將新金鑰新增到 SSH agent 中

因為作業系統預設只讀取私鑰 id_rsa，為了讓 SSH 識別新的私鑰，需將其新增到 SSH agent 中：

```
ssh-add ~/.ssh/金鑰名稱
```

如果出現 Could not open a connection to your authentication agent 的錯誤，就試著用以下命令：

```
ssh-agent bash
ssh-add ~/.ssh/金鑰名稱
```

</br>

{{< notice warning >}}
重啟電腦後都需要重新 ssh-add。因為這個命令不會永久性的記住私鑰。使用 ssh-add 會把指定的私鑰新增到 ssh-agent 所管理的一個 session 當中。  
而 ssh-agent 是一個用於儲存私鑰的臨時性的 session 服務，也就是說當你重啟之後，ssh-agent 服務便會重置，session 會話也會失效。
{{< /notice >}}

## 解決每次重啟都要 ssh-add 的問題

在 git 安裝目錄下的 etc/bash.bashrc 文件中末加入

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/金鑰名稱
```

## 配置多個 ssh-key

修改 `~/.ssh/config` 文件

```
#default github
Host github.com
HostName github.com
IdentityFile ~/.ssh/id_rsa

Host github_second
HostName github.com
IdentityFile ~/.ssh/id_rsa_second
```

其中 `Host` 為 `HostName` 的別名。

## 測試連線

```
ssh -T git@[Host]
```

出現

```
Hi xxx! You’ve successfully authenticated, but GitHub does not provide shell access.
```

代表連線成功

## 在專案下設配置使用者

先取消 global

```
git config --global --unset user.name
git config --global --unset user.email
```

設置 repo 自己的 user & email

```
git config  user.email "xxxx@xx.com"
git config  user.name "xxxx"

```

clone 遠端資料

```
git clone git@[Host]:UserName/repositoryName
```

git push

```
git remote add origin git@[Host]:UserName/repositoryName.git
git add -am "commit msg"
git push origin master
```

## Reference

[1] https://segmentfault.com/q/1010000000835302  
[2] https://blog.csdn.net/EsonJohn/article/details/79134665\  
[3] http://blog.lessfun.com/blog/2014/06/11/two-github-account-in-one-client/
