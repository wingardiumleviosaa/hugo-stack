---
title: "[Windows] 使用 winrm 遠端連線到其他主機"
tags:
  - Windows
  - Powershell
categories:
  - SRE
date: 2023-01-12 11:06:00
slug: win10-winrm-to-remote-host
---

<!--more-->

## 遠端機器設定

winrm service 預設都是未啟用的狀態，先檢視狀態；如無返回資訊，則是沒有啟動；

```powershell
winrm enumerate winrm/config/listener
```

針對 winrm service 進行基礎配置：

```powershell
winrm quickconfig
```

重新檢視 winrm service listener:

```powershell
winrm e winrm/config/listener
```

為 winrm service 配置 auth:

```powershell
winrm set winrm/config/service/auth '@{Basic="true"}'
```

為 winrm service 配置加密方式為允許非加密：

```powershell
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```

## 本地機器設定

如果要用 powershell 連的話，本地端也需要設置 trustedhosts 的 value

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
```

如果丟 Access Denied 的錯誤的話，則使用 Administrator 身分開啟 powershell 並設置

```powershell
Set-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -Name LocalAccountTokenFilterPolicy -Value 1 -Type DWord
```

![](https://imgur.com/f7YJeHd.png)

## 使用 Invoke-Command 連線下指令

```powershell
Invoke-Command -ComputerName <遠端系統名稱> -ScriptBlock { <遠端指令> } -credential <遠端系統使用者帳戶名>
```

將 `<遠端系統名稱>` 替換為遠端系統的名稱或 IP 位址，並將 `<遠端指令>` 替換為要在遠端系統上執行的 PowerShell 指令。指令應該放在大括號 `{}` 內，以便於形成腳本塊。最後帶上要使用哪個使用者帳戶執行，會跳出視窗輸入帳密資訊。

## Reference

- https://dotblogs.com.tw/momoBear/2018/01/04/160854
- https://social.technet.microsoft.com/Forums/Azure/en-US/b853e77c-7c9a-4231-a32b-c63727ec5868/access-denied-when-trying-to-set-trusted-hosts-for-psremoting?forum=winserverpowershell
