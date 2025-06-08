---
title: "[Windows] 使用 openssh 連線到其他主機"
tags:
  - Windows
categories:
  - SRE
date: 2023-01-10 10:13:00
slug: win10-openssh
---

<!--more-->

在連線機器上的在 Windows 選用功能下安裝 OpenSSH Server

![](https://imgur.com/yuX5J34.png)

或是使用 powershell 設定

```powershell
Get-Service -Name *ssh*
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

便可以使用 22 端口連線至機器上

```powershell
ssh -p 22 Administrator@192.168.1.90
```
