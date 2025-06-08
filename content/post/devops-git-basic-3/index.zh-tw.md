---
title: "[Git] Basic Git (3) 一些狀況劇"
author: Ula Hsieh
tags:
  - Git
categories:
  - DevOps
date: 2020-06-13 10:45:00
slug: git-basic-3
---

## 想改最後一次的 commit message

在本地端 git commit 後發現 commit message 打錯了，只要下 `$ git commit --amend`即可進入 vim 編輯器做修改。

<!--more-->

![](https://imgur.com/7uZHzPo.png)

<table><tr><td bgcolor=#FDEDEC>
請注意，如果已經 commit 而且又 push to remote 了，那就乖乖認命吧，這種情形下你在 local 端改的話可能會造成其他人的困擾。最好的方法還是 push 之前先檢查一下，避免錯的東西被放到遠端。
</td></tr></table>

## commit 後但後悔了

使用 `$ git reset HEAD^ (--soft / --hard / --mixed)`

|       名詞        |           解釋            |
| :---------------: | :-----------------------: |
|       head        |         所在位置          |
|         ^         |          上一個           |
|       index       | 變更狀態紀錄 (git status) |
| working directory |         工作目錄          |

|      mode      |  head   |   index   | working directory | 說明                                                              |
| :------------: | :-----: | :-------: | :---------------: | :---------------------------------------------------------------- |
|      soft      | changed | unchanged |     unchanged     | 僅移除 commit 變成新版未 commit，內容仍是新版的。                 |
| mixed(default) | changed |  changed  |     unchanged     | index 移除 staged 標記，變成 Modifiedor Untracked，內容是新版的。 |
|      hard      | changed |  changed  |      changed      | 回到上一版版本，完全移除，內容及狀態皆是上一版。                  |

[1]

## 改了檔案還沒 commit 但想復原

用 `$ git restore <file>` 回復，或是 `$ git restore .`回復所有檔案
或是舊 command `$ git checkout -- <file>`

![](https://imgur.com/3yBqfnp.png)

## 想改 branch 名稱

`$ git branch -m <新名稱>`

![](https://imgur.com/GxP9UtR.png)

## 想把遠端的 branch 抓下來

設目前本地端無任何 branch，可以直接下 `$ git checkout <remote-branch-name>`。

![](https://imgur.com/drkR4Vc.png)

## 想在 commit 前做一些判斷 (git hooks)

在 `.git` 資料夾下有 `hooks` 資料夾，裡面存放了一些 shell script，可以讓使用在在針對某些狀況下設置一些判斷，然後 git 做一些反應，比如說在 commit 或是 push 前檢查是否有放帳號密碼等資訊，然後停止或允許動作。

![](https://imgur.com/noHMLbm.png)

## Source

[1] https://ithelp.ithome.com.tw/articles/10187303
[All] 大部分的內容皆來自 [Lidemy](https://lidemy.com/) [GIT101] 的課堂筆記
