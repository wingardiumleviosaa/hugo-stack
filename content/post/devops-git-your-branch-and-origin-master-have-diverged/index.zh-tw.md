---
title: "[Git] Your branch and 'origin/master' have diverged"
tags:
  - Git
categories:
  - DevOps
date: 2022-03-29 20:19:00
slug: git-your-branch-and-origin-master-have-diverged
---

在 git status 的時後發現本地端跟遠端倉庫 commit 分岔 (diverged)，代表本地所在的分支跟遠端倉庫的分支各走各的路。

<!--more-->

## 問題

```bash
git status
  On branch master
  Your branch and 'origin/master' have diverged,
  and have 1 and 1 different commits each, respectively.
    (use "git pull" to merge the remote branch into yours)

  nothing to commit, working tree clean
```

## 原因排查

查看各自的 commit log，在本地端的 log 中沒有發現 origin/master 遠端倉庫的 master HEAD，所以代表分支已經分歧。

```bash
git log --oneline origin/master
  45b6977 (origin/master) v1.1 modified return body
  2714cef add changelog
  4a19b17 ula modified reverse rule
  92d48af origin 'reverstapi' version
git log --oneline
  733e302 (HEAD -> master) v1.2 rebase from v1 and modified the sql command
  2714cef add changelog
  4a19b17 ula modified reverse rule
  92d48af origin 'reverstapi' version
# 利用 git cherry origin/master 來看遠端與本地端 commit 的差別
git cherry origin/master
  + 733e302d65419093b386de9b286c96eecaa95200
# 顯示有一個版本忘了提交
```

## 同步遠端與本地分支

有以下四種狀況，主要分為合併兩分支與忽略某一方分支。合併又分為兩種 merge & rebase；而忽略其中一方分支，則要先確定哪一邊的分支是對的，如果是多人協做的環境，通常遠端倉庫是對的情況下，我們就是處理本地端的部分就好。

### 分支合併(merge)

1. 更新 remote branch，fetch 會去讀取 remote repo 的內容，並且更新 remote branch 的內容，但不會修改本地端分支。

```bash
git fetch origin master
```

2. 如同 status 提示，使用 git pull 來 merge 遠端的分支

```bash
git pull origin master
# 同等於 git merge remotes/origin/master
```

![](https://imgur.com/8MhgIh5.png)

git pull 會幫我們做 merge，若沒有發生版本衝突，就會直接 Fast-Forward，會直接修改 master branch 的 HEAD 指向位置，直接移動到遠端 master branch 的 HEAD 也就是 FETCH_HEAD。但若發生了版本衝突的時候，就需要提交 Merge Patch，但若改動很少，使用 merge 會產生多餘且沒意義的 Merge Patch，較不建議使用。

### 分支合併(rebase)

1. 更新 remote branch

```bash
git fetch origin
```

2. git rebase，重新修改特定分支的「基礎版本」，把另外一個分支(remote master)的變更，當成我這個分支(local master)的基礎。

```bash
git pull --rebase origin/master
# 同等於 git rebase remotes/github/master
```

3. master 分支同步成功之後，就可以把後來 master 新修改的 commit push 出去了

```bash
git push origin master
```

### 處理本地端倉庫(忽略本地端倉庫的修改)

先使用 reset 回到與遠端倉庫一樣的 commit (2714cef)

```bash
git reset --hard 2714cef
```

回到了與遠端倉庫一樣擁有的 commit，但這個 commit 並不是 origin/master 倉庫最新的 commit，git 告訴我們落後了 1 個 commit

```bash
git status
 On branch master
 Your branch is behind 'origin/master' by 1 commits, and can be fast-forwarded.
   (use "git pull" to update your local branch)

 nothing to commit, working tree clean
```

將遠端倉庫拉回來補上這些漏掉的 commit 就解了

```bash
git pull origin master
git status
 On branch master
 Your branch is up to date with 'origin/master'.

 nothing to commit, working tree clean
```

### 處理遠端倉庫(忽略遠端倉庫的修改)

{{< notice warning >}}
基本上，用此方法強制 push 本地端的更新到遠端，會使遠端修改丢失，一般是不可取的，尤其是多人協作開發的時候。
{{< /notice >}}

但我這次遇到的狀況恰好就是要蓋掉遠端的更新。在 master branch，強制更新 remote master branch。  
\*\*需要先解除 remote master branch 的 protect (Github or Gitlab 的遠端倉庫上 Settings / Repository / Protected branches)

```bash
git push origin master -f
  Enumerating objects: 13, done.
  Counting objects: 100% (13/13), done.
  Delta compression using up to 8 threads
  Compressing objects: 100% (7/7), done.
  Writing objects: 100% (7/7), 905 bytes | 905.00 KiB/s, done.
  Total 7 (delta 5), reused 0 (delta 0)
  To ssh://git.nexmasa.com:10022/root/reverseapi.git
   + 45b6977...733e302 master -> master (forced update)
```

完成後，查看兩邊的 git log，可以發現遠端的 commit 強制與本地端的 master 同步了。

```bash
git log --oneline origin/master
  733e302 (HEAD -> master, origin/master) v1.2 rebase from v1 and modified the sql command
  2714cef add changelog
  4a19b17 ula modified reverse rule
  92d48af frank origin 'reverstapi' version
git log --oneline master
  733e302 (HEAD -> master, origin/master) v1.2 rebase from v1 and modified the sql command
  2714cef add changelog
  4a19b17 ula modified reverse rule
  92d48af frank origin 'reverstapi' version
```

## 補充(revert)

git revert 命令意思是撤銷某次提交。它會產生一個新的提交，雖然程式碼回退了，但是版本依然是向前的，所以，當你用 revert 回退之後，其他開法者 pull 遠端倉庫之後，他們的程式碼也自動的回退了。通常比較適用於已經 push 出去的 Commit，或是不允許使用 Reset 修改歷史紀錄的指令的場合。

```sh
# 撤銷最近一次提交
git revert HEAD
# 撤銷最近 2 次提交，注意：數字從 0 開始
git revert HEAD~1
# 撤銷指定的提交
git revert <commitID>

# 更新遠端
git push
```

## Reference

- https://www.nvda.org.tw/discussion/ui=100204tm=1973254239
- https://backlog.com/git-tutorial/tw/stepup/stepup3_2.html
- https://zlargon.gitbooks.io/git-tutorial/content/remote/sync.html
