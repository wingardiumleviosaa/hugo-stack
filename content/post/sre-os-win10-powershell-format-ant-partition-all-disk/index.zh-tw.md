---
title: "[PowerShell] 一次分割所有硬碟"
tags:
  - PowerShell
  - Windows
categories:
  - SRE
date: 2022-11-19 21:18:00
slug: powershell-partition-and-format-all-disk-at-once
---

## Get-Disk

取得當前連接到 win10 的所有硬碟

```
Get-Disk
```

<!--more-->

</br>

![](https://imgur.com/Ksixr3v.png)

## 硬碟初始化

將硬碟設置為 online 以及關閉 readonly，並初始化所有硬碟指定分割區樣式為 GPT（GUID Partition Table）。

```powershell
try {
  Get-Disk | ?{$_.number -ne 0}| Set-Disk -IsOffline $False
  Get-Disk | ?{$_.number -ne 0}| Set-Disk -isReadOnly $False
  Get-Disk | ?{$_.number -ne 0}| Initialize-Disk -PartitionStyle GPT
}catch{
  Write-Host $_.Exception.Message
}
```

## 磁碟分割

依序分割磁碟並指派磁碟槽代號，最後格式化

```powershell
try {
  # 為所有硬碟建立分區並自動指派硬碟代號以及使用最大的硬碟大小
  Get-Disk | ?{$_.number -ne 0}| New-Partition -AssignDriveLetter -UseMaximumSize
  # 取得所有硬碟的分區並格式化
  Get-Disk | ?{$_.number -ne 0}| Get-Partition |?{$_.type -like "Basic"}| Format-Volume -Confirm:$false
}catch{
  Write-Host $_.Exception.Message
}
```

其中 Format-Volume 使用預設的檔案系統 NTFS，還有其他可能的值：

- NTFS：預設檔案系統，提供了對大型檔案和分割區的支援，並具有高度的可靠性和安全性。
- FAT32：與 Windows 和其他操作系統的相容性較好，但不支援單個檔案超過 4GB。
- exFAT：與 Windows 和其他操作系統的相容性較好，支援大型檔案和分割區，但不如 NTFS。

## Reference

- http://www.alexandreviot.net/2015/05/02/powershell-how-to-format-all-disks/
