---
title: "[Git] Basic Git (2) - Branch & GitHub"
author: Ula Hsieh
tags:
  - Git
categories:
  - DevOps
date: 2020-06-13 09:18:00
slug: git-basic-2
---

## branch 概念

一般在線性開發時會是以下這樣：

![](https://imgur.com/yxIhSIN.png)

當在開發新功能時，發現當下開發的版本有舊有的 bug，此時如果一邊開發一邊改 bug 可能會導致產出的東西有衝突。  
而如果引入分支 branch，讓開發新功能以及 debug 兩邊各自獨立完成，而後再進行合併，就可以把工作乾淨地切割開來。目的是為了保持主枝幹的穩定，因為在開發新功能的時候不可能隨時保持穩定狀態，所以在確定穩定前都不會影響到主幹。

![](https://imgur.com/KGfAOUW.png)

<!--more-->

### git branch -v

可以查看目前有哪些 branch，顯示方式為 `<branch name> <latest commit version> <commit message>`，亮綠燈的代表目前工作目錄所在的分區。

![](https://imgur.com/t0qVj6k.png)

### git branch

`$ git branch <branchname>` 可依目前 branch 為基準複製出一個新的分支。

![](https://imgur.com/9B0IRri.png)

### git checkout

`$ git checkout <branchname>` 可以將目前工作目錄切換到指定 branch 下，可以把 checkout 想像成移到該資料夾下的感覺。

![](https://imgur.com/Er1RrvV.png)

### git merge

`$ git merge <branchname>` 可以把指定的 branch 合併到當下的工作目錄來。如下圖，在 new-feature branch 下修改了 hello.js 的檔案，回到 master branch，先 cat 確認 hello.js 內容是舊的，再下 merge command 將 new-feature branch 合併到 master 後，就能發現 master 的檔案也更新了！

![](https://imgur.com/p80UcmR.png)

### git branch -d

`$ git branch -d <branchname>` ，-d 為 --delete，可刪除該 branch。通常在開發完該 branch 並合併進主支線後，就可以刪除分支以保持乾淨的專案內容。

![](https://imgur.com/eJETGfT.png)

### 處理 merge 後的 conflict

如果 branch 在合併的時候有檔案有衝突，意即同個檔案的檔案內容的同一行不一樣，git 不曉得該以哪一個為準，則會在 merge 時出現下面的訊息：

![](https://imgur.com/EpEnTVJ.png)

須先查看發生衝突的檔案

![](https://imgur.com/9IryZEM.png)

Git 把有衝突的段落標記出來了，上半部是 HEAD，也就是目前所在的 master 分支，中間是分隔線，接下是 new-feature 分支的內容。請做修改並儲存檔案，最後再次 `$ git commit -am 'reslove the conflict'`，結束這回合！

## Git v.s. Github

Github 就是一個幫你把 git repository 具象化的一個網站，一個有 UI 的 git server。

![](https://imgur.com/5q7iRqR.png)

### git push (from local)

![](https://imgur.com/pl7p1VW.png)

在 github 新增 repo 後回到本地端，要將本地端的 repo push 上去

```
$ git remote add origin https://github.com/ulahsieh/gitTest.git
$ git push -u origin master
```

第一個指令代表加入一個遠端的 repository 代號是 `origin`；第二行指令是把目前的 branch push 到 origin 的 master branch。

```
-u 代表 -set-upstream，設定 upstream 可以使分支開始追蹤指定的遠端分支，只要做過一次 git push -u <remote name> <branch name>，並且成功 push 出去；當下本機端的 branch 就會自動與遠端的 <remote name>/<branch name> 分支設定好 upstream 連結，之後要上傳分支時，就只需要透過簡單的 git push 指令就可以了。
```

![](https://imgur.com/IfB7ZK0.png)

就能看到 github 上的 repo 與 local 同步

![](https://imgur.com/BLWrMNa.png)

當在本地端做 git commit 後，再下 git push 就可以在遠端分支更新 (因為前一次 push 有加 -u)。

![](https://imgur.com/zUU1ljm.png)

### push 其他分支

`$ git push origin <branch name>`，把當下分支 push 上 remote repository。

![](https://imgur.com/ZRnBUq1.png)

可以發現到 github 能切換剛剛 push 的分支

![](https://imgur.com/voSzmUU.png)

### git pull (pull remote repo at local)

github 可以在線上編輯 code，或是共同協做的時候，其他人做過更動想要同步在本地端時，下 `$ git pull origin master`，從遠端 origin pull master branch 下來。

{{< notice warning >}}
請注意，如果在遠端做更新後，本地端一定要先 git pull 將最新版同步，才能再做 git push!
{{< /notice >}}

![](https://imgur.com/lBtK1xX.png)

如果 git pull 後發現遠端的與本地端有檔案的內容有衝突，那在 pull 結束後會顯示 conflict，改掉衝突再 commit & push 即可。

### Pull request

github 上的 `Pull Request` 用於合併分支，通常在 github 做 merge 方便又可以很快追蹤到兩個 branch 的差異，所以通常在做 merge 的時候都會在 github 做。  
首先在本地的 new-feature branch 建一個新的 newfile.js， git add & commit 後再 git push 上遠端的 new-feature repo。

![](https://imgur.com/HPm3L42.png)

到 github 點選 `Compare & pull request` 進行合併，

![](https://imgur.com/umOt0Q1.png)

接著一步一步做下來即可。完成 merge 後，就可以直接在 github 上刪除該 branch。

![](https://imgur.com/OomNCs6.png)

最後記得回到 local 同步 repo，`$ git pull origin master` & `$ git branch -d new feature`

![](https://imgur.com/pfplGn1.png)

### Pull request 發生衝突

在 pull request 如果有 `can't automatically merge` 訊息出現的話代表有 conflict 出現，

![](https://imgur.com/6mTgbAd.png)

改掉衝突後

![](https://imgur.com/qSRL0tO.png)

![](https://imgur.com/Lubw3xf.png)

就可以順利 commit 了

![](https://imgur.com/tTtWGjJ.png)

### git clone

除了可以 pull 自己的 github repo 外，也可以下載其他人的 repository 到自己的本地端。按下 `clone or download` 即可使用 zip 下載，或是複製 git clone 語法到 git bash 裡用 command line 下載。

![](https://imgur.com/3Dkr3RS.png)

### Fork

但是如果想要修改從別人 github 的 repository 下載下來的 repo 到本地然後又想 git push 到自己的 github 的話是不允許的。必須按右上角的 `Fork`，先在 github 上將這個 repo fork 到自己的 github，

![](https://imgur.com/15OAr1b.png)

然後從自己的 repo git clone 下來，才可以做修改後再 push 上自己的 github。
{{< notice warning >}}
請注意，用 fork 的情況是因為沒有該原 repo 的權限 (permission deny)，如果自己本身有被 repo 的作者開啟修改的權限的話，就可以直接在本地端 git push 上他的 repo
{{< /notice >}}

## source

[all] 大部分的內容皆來自 [Lidemy](https://lidemy.com/) [GIT101] 的課堂筆記
