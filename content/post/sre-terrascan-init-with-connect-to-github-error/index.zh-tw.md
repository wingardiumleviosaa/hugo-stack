---
title: "[Terrascan] init with connect to github error"
tags:
  - Terraform
categories:
  - SRE
date: 2023-03-14 21:10:00
slug: terrascan-init-with-connect-to-github-error
---

## TL;DR

在使用 terrascan 當作 IAC 語法檢查工具時，發現跑在公司內部環境的 gitlab pipeline 都會卡在 terrascan init 的錯誤。

<!--more-->

## 問題描述

Dockerfile 是透過 Gitlab Pipeline 幫忙打包的，在 Job 中跑 git clone 到公司內部存放 OPA Policy 的 Repo 皆沒問題，但是 Job 跑到 terrascan init 時總是會報 `error	cli/init.go:42	failed to initialize terrascan. error : could not connect to github.com` 的錯誤。

![](https://imgur.com/cQTgKQi.png)

## 環境

- terrascan: 1.16.0

## Code Tracing

因為在前面有用 git clone 測試連結到目標 repo 是通的，所以就把問題縮小在是 terrascan 在 init 時做了什麼動作。可以在 terrascan init 時加上 `-l debug` lebel 幫助排查問題。

![](https://imgur.com/4zqOR4S.png)

```go
func DownloadPolicies() error {
	accessToken := config.GetPolicyAccessToken()
	policyBasePath := config.GetPolicyBasePath()

	zap.S().Debug("downloading policies")
	zap.S().Debugf("base directory path : %s", policyBasePath)

	err := os.RemoveAll(policyBasePath)
	if err != nil {
		return fmt.Errorf("unable to delete base folder. error: '%w'", err)
	}

	if accessToken == "" {
		return downloadDefaultPolicies(policyBasePath)
	}

	return dowloadEnvironmentPolicies(policyBasePath, accessToken)
}
```

init 時會去 clone policy，如果沒有額外指定，會去抓取 tenable/terrascan 官方的 repo，而在公司環境雖然是指定內部的 repo，不過該 repo 也是在內網中公開的，所以去到的是 downloadDefaultPolicies function。

```go
func downloadDefaultPolicies(policyBasePath string) error {
	if !connected(terrascanReadmeURL) {
		return errNoConnection
	}

	repoURL := config.GetPolicyRepoURL()
	branch := config.GetPolicyBranch()

	zap.S().Debugf("policy directory path : %s", repoURL)
	zap.S().Debugf("policy repo url : %s", repoURL)
	zap.S().Debugf("policy repo git branch : %s", branch)
	zap.S().Debugf("cloning terrascan repo at %s", policyBasePath)

	// clone the repo
	r, err := git.PlainClone(policyBasePath, false, &git.CloneOptions{
		URL: repoURL,
	})

	if err != nil {
		return fmt.Errorf("failed to download policies. error: '%w'", err)
	}

	// create working tree
	w, err := r.Worktree()
	if err != nil {
		return fmt.Errorf("failed to create working tree. error: '%w'", err)
	}

	// fetch references
	err = r.Fetch(&git.FetchOptions{
		RefSpecs: []gitConfig.RefSpec{"refs/*:refs/*", "HEAD:refs/heads/HEAD"},
	})
	if err != nil {
		return fmt.Errorf("failed to fetch references from git repo. error: '%w'", err)
	}

	// checkout policies branch
	err = w.Checkout(&git.CheckoutOptions{
		Branch: plumbing.ReferenceName(fmt.Sprintf("refs/heads/%s", branch)),
		Force:  true,
	})
	if err != nil {
		return fmt.Errorf("failed to checkout git branch '%s'. error: '%w'", branch, err)
	}

	return nil
}

func connected(url string) bool {
	_, err := http.Get(url)
	return err == nil
}
```

這邊就直接看到原因了，在 function 的一開始會去做 `connected(terrascanReadmeURL)`，而這個丟進去的參數並不是定義在 config 檔的 repo_url，而是 terrascanReadmeURL (https://raw.githubusercontent.com/tenable/terrascan/master/README.md)，所以在內網環境因為公司的 Kubernetes 有設防火牆，所以直接被擋掉了。

## 解決方法

捨棄 terrascan init，直接自己用 git clone 並把 clone 下來的 repo 放到 terrascan 的指定資料夾 `~/.terrascan/`，跳過 source code 的 bug。
