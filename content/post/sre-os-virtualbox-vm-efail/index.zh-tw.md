---
title: VirtualBox 安裝 VM 錯誤 (E_FAIL 0x80004005) 之解決
tags:
  - VMWare
categories:
  - DevOps
date: 2022-02-21 19:14:00
slug: virtualbox-efail-error
---

在 Win10 使用 VirtualBox 安裝 VM 時遇到下述錯誤，紀錄一下解決方式。

<!--more-->

## Error:

```
Failed to open a session for the virtual machine xxx.
Call to WHvSetupPartition failed: ERROR_SUCCESS (Last=0xc000000d/87) (VERR_NEM_VM_CREATE_FAILED).
Result Code: E_FAIL (0x80004005)
Component: ConsoleWrap
Interface: IConsole {872da645-4a9b-1727-bee2-5585105b9eed}
```

## Reason

許多虛擬化應用程式依賴處理器上的硬體虛擬化擴充，包括 Intel VT-x 和 AMD-V，一次僅能允許一個軟體使用。若要使用其他虛擬化軟體如 VirtualBOX 或 VMware，必須停用 Hyper-V 虛擬機器監控程式、裝置防護和認證防護。

## Solution:

```
Windows Features
----------------------
Disabled -> Hyper-V
Enabled  -> Virtual Machine Platform
Enabled  -> Windows Hypervisor Platform
Disabled -> Windows Sandbox

Elevated Powershell/Cmd
-------------------------------
bcdedit /set hypervisorlaunchtype off

BIOS
-----
Enabled  -> Virtualization Technology (VTx)
Enabled  -> Virtualization Technology for Directed I/O (VTd)
Disabled -> HP Hypervisor
```

Other Windows Features (Possibly irrelevant, can view img below)

![](https://i.imgur.com/NgDGQvW.png)

## Reference

https://forums.virtualbox.org/viewtopic.php?f=6&t=93443
https://docs.microsoft.com/zh-tw/troubleshoot/windows-client/application-management/virtualization-apps-not-work-with-hyper-v
