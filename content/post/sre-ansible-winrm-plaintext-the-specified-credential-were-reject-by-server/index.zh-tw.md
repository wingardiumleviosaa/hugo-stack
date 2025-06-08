---
title: >-
  [Ansible] winrm plaintext: the specified credentials were rejected by the
  server
tags:
  - Ansible
categories:
  - SRE
date: 2022-09-06 21:11:00
slug: ansible-winrm-plaintext-credentials-error
---

紀錄 Ansible 使用 Add host 動態新增 Windows 控制節點時，遇到的 winrm 問題及解決方法。

<!--more-->

## 問題

在只有配置以下參數的情況下連結遠端 windows server，會出現標題寫的錯誤訊息。

```yaml
ansible_user: user@DOMAIN.COM
ansible_password: password
ansible_connection: winrm
ansible_ssh_port: 5986
```

## 解決方式

1. 確認 remote windows 的 winrm 模組是否有配置

![](https://imgur.com/fbeVHKp.png)

2. Ansible host 配置

```yaml
- name: Add the host
      add_host:
        name: win2019
        ansible_connection: winrm
        ansible_port: 5985
        ansible_host: "10.37.39.222"
        ansible_user: "Administrator"
        ansible_password: "Passw@ord"
        ansible_winrm_transport: ntlm
        ansible_winrm_server_cert_validation: ignore
      no_log: true
```
