---
title: "[Git] Basic Git (1) - Intro & Init, add, commit, status... "
tags:
  - Git
categories:
  - DevOps
date: 2020-06-13 07:15:00
slug: git-basic-1
---

## 前言

在沒有版本控制系統前，如果有一個常常在修改的檔案，又想保留每個版本狀態時，我們常會在編輯檔案前複製一個備份，時間久了就會變得非常不便且難以維護，有可能造成命名混亂，也很難比較各版本間的差異，尤其是在多人協作的狀況下時還可能會發生衝突。

<!--- more --->

## 版本控制概念－用一般資料夾闡述

如果是以資料夾去做版控的概念會是如下

<table><tr><td bgcolor=AliceBlue>
  1. 需要新版本：開一個新的資料夾</br>
  2. 不想加入版控：不要加入資料夾</br> 
  3. 避免版本號衝突：用看似亂數的東西當作資料夾名稱</br>
  4. 知道資料夾順序：用一個檔案來紀錄</br>
  5. 知道最新版本：用一個檔案來存</br>
</td></tr></table>

## GIT 是什麼

git 就是個幫你做版控的程式，不用像上面那樣做很多動作！

## 開始使用

### git init

進入要做版控的資料夾/專案，並下 `$ git init` 做初始化。初始化完成後可以看到目錄下多了一個 `.git` 的隱藏子資料夾，其中包含 Git 所有必需的倉儲檔案，也就是 Git 倉儲的骨架。

![](https://imgur.com/d3kVM31.png)

### git status

用於查看目前工作目錄 (working directory) 的檔案狀態，以下是檔案可能會有的四種狀態：

![](https://imgur.com/0n6e7EE.png)

- **未追蹤 (Untracked)**：沒有被 GIT 所追蹤控管的檔案，如新增的檔案。
- **已更改 (Modified)**：已提交版本後，又再次修改的檔案。
- **等待提交 (Staged)**：在工作目錄 (WD) 的檔案執行 git add 後，會放在暫存區 (Stage) 等待提交。
- **已提交 (Committed)**：在暫存區的檔案執行 git commit 後，檔案便置於儲存區 (Repo)，這些放在儲存區的檔案即是已提交的狀態。

第一次下 `$ git status` 可以發現目前目錄下的檔案狀態皆為  <font color=#FF5959>Untracked files</font>，表示這些是全新的檔案，沒有被加入版控。

![](https://imgur.com/NssQJIh.png)

### git add

輸入 `$ git add <filename>` 可將檔案指定加入版控 / 暫存區 (Stage)；  
輸入　`$ git add .` 可一次加入 <span class="dotunderletter">所有</span> 檔案
再查看狀態可以看到檔案狀態變為 <font color=green>Changes to be committed</font>。  
![](https://imgur.com/l10gh8Z.png)

### git commit

輸入`$ git commit` 將建立新的版控，將放在暫存區的檔案放入 repository
如果單獨輸入`$ git commit`，則會先進入 vim 編輯器，要求在編輯器輸入 commit 訊息。  
或是下`$ git commit -m "commit message"` 直接在後面帶訊息。如果沒有 commit 訊息的話則會 commit 失敗。

![](https://imgur.com/OAxsNhn.png)

### git rm

`$ git rm --cached <filename>` 可以將檔案從 staged 或是 committed 狀態移除版控。

![](https://imgur.com/QxmpZZt.png)

### git log

輸入 `$ git log` 可以查看所有提交紀錄。

![](https://imgur.com/GJawPx4.png)

新增參數 --oneline `$ git log --oneline`，可以顯示較為簡短的 git log。

![](https://imgur.com/r7bmS5L.png)

### git commit -am

在修改已經 commit 的檔案後，可以發現檔案變成 <font color=#FF5959>Changes not staged for commit</font> 的 <font color=#FF5959>modified</font> 狀態。

![](https://imgur.com/UIEt4tF.png)

此時可以下 `$ git add .` 並下 `$ git commit -m "msg"`。或是直接下 **`$ git commit -am "msg"`** 將兩個指令合併。 `-a` 參數表示 `--all` (git add --all)，會把 <span class="dotunderletter">已修改過</span> 的檔案加入 staged 區。

{{< notice warning >}}
請注意，使用 git commit -am 不會包含新增的檔案 newfile，需要先 git add 再 git commit 個別下。
{{< /notice >}}

p.s. git add 後檔案狀態會變成 <font color=green>Changes to be commited (Modified)</font>

![](https://imgur.com/lCUEpVZ.png)

### git checkout

使用 `$ git check out <版本>` 可以回到指定的版本時的狀態。例如下圖回到了 first commit 當下。

![](https://imgur.com/T95UVYe.png)

### .gitignore

新增 .gitignore，若有檔案名稱寫入該檔案則會被 git 忽略，專門放與專案沒什麼關係、不須版控也可以的檔案，但 .gitignore 檔案本身會加入版控，要告知其他人什麼檔案不在版控中。

![](https://imgur.com/iGu6x8V.png)

### git diff

`$ git diff` 可以查看這次要 commit **前**與上一個版本的差別。

![](https://imgur.com/OB2WhIl.png)

## source

[all] 大部分的內容皆來自 [Lidemy](https://lidemy.com/) [GIT101] 的課堂筆記
