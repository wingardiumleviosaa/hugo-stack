---
title: "[Ansible] Windows Update with Authenticated Proxy"
tags:
  - Ansible
categories:
  - SRE
date: 2022-10-23 18:59:00
slug: ansible-windows-update-with-auth-proxy
---

## 背景

在透過 ansible windows update module 更新 windows 時，都會因為 proxy 沒有設置而失敗。

<!--more-->

## 解決方式

```yaml
- name: Configure IE proxy settings to apply to all users
  ansible.windows.win_regedit:
    path: HKLM:\SOFTWARE\Policies\Microsoft\Windows\CurrentVersion\Internet Settings
    name: ProxySettingsPerUser
    data: 0
    type: dword
    state: present

- name: Configure IE to use a specific proxy per protocol
  community.windows.win_inet_proxy:
    auto_detect: no
    proxy: whqproxys.abc.com:8080

- name: Set credential to use for proxy auth
  community.windows.win_credential:
    name: whqproxys.abc.com
    type: generic_password
    username: "{{ proxy_user }}"
    secret: "{{ lookup('env', 'VC_PASSWORD') }}"
    state: present
  become: yes
  become_user: Administrator
  become_method: runas

- name: Set the proxy to be able to run  Windows Updates
  ansible.windows.win_command:
    cmd: netsh winhttp import proxy source=ie

- name: Search-only, return list of found updates (if any), log to txt file
  ansible.windows.win_updates:
    category_names: "*"
    state: searched
    log_path: C:\Users\Administrator\Desktop\ansible_update_list.txt

- name: Windows Update which may take a significant amount of time to complete
  ansible.windows.win_updates:
    category_names: "*"
    state: installed
    reboot: yes
    log_path: C:\Users\Administrator\Desktop\ansible_update_log.txt
```

## 補充

[Ansible win_inet_proxy 的文件](https://docs.ansible.com/ansible/latest/collections/community/windows/win_inet_proxy_module.html#ansible-collections-community-windows-win-inet-proxy-module) 在設定 IE http proxy 時是透過 win_http_proxy 的模組，但該作法好像失敗，還是需用 `netsh winhttp import proxy source=ie` 才可正確應用。

```yaml
# 失敗
- name: Import IE proxy configuration to WinHTTP
  community.windows.win_http_proxy:
    source: ie
```

## Reference

- [感謝極為稀有的 serverfault 上的問題 T_T](https://serverfault.com/questions/1107422/windows-update-through-explicit-squid-proxy-0x80072ee6)
