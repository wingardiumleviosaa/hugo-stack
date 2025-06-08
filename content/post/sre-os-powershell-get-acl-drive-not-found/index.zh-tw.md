---
title: "[PowerShell] Get-Acl Drive Not Found Error"
tags:
  - PowerShell
categories:
  - SRE
date: 2022-09-06 21:07:00
slug: powershell-get-acl-drive-not-found-error
---

紀錄一下在 windows10 環境下無法正常使用 get-acl 訪問 active directory 網域服務下的資源的問題與解決方式。

<!--more-->

## 問題描述

透過在 Azure 上建立一台 Windows (21h2)，並建立 peering 到公司內部的 hub，使這台 VM 可以用公司內部的 IP 連線並加入公司網域，但要透過這台 VM 執行 active directory module 的系列操作時，會遇到以下問題：

1. 首先在 import ActiveDirectory 時會有以下錯誤

![](https://imgur.com/Aknk4vv.png)

2. 在使用 get-acl cmdlet 時會遇到以下錯誤

![](https://imgur.com/KMZwtSX.png)

## Solution

在 import active directory module 後，針對找不到 drive 的錯誤去新增 PS Drive。
首先在目前環境下使用 `Get-PSDrive` 查看目前掛載的目錄，的確在目前環境中沒有任何 AD 為命名的 drive。

![](https://imgur.com/OrZ6D5k.png)

用以下方式新增，並切換目錄:

```shell
New-PSDrive -Name AD -PSProvider ActiveDirectory -Root "OU=CCOE,OU=Security_Group,OU=Group_Object,DC=whq,DC=wistron" -server "hqnhudc1.whq.wistron"

set-location AD:\
```

完成後便可以正常使用 get-acl 指令去取得指定路徑的存取權限：

```
get-acl -path 'Microsoft.ActiveDirectory.Management.dll\ActiveDirectory:://RootDSE/CN=COGTESTAD,OU=CCOE,OU=Security_Group,OU=Group_Object,DC=whq,DC=wistron'
```

注意 path 後帶的目錄位址不能直接使用 distinguished name，而是完整的路徑

![](https://imgur.com/Mu8Z0nS.png)

## 補充說明

測試後，該 new drive 的動作獨立於每個 powershell session，所以必須在每次開啟 terminal 時都動作，如果想要在每次開啟後都自動加載的話，請參考 `PowerShell Profile` 的設定檔設定。

## Reference

- https://www.reddit.com/r/PowerShell/comments/38o6me/how_do_i_save_a_newpsdrive_psprovider/
