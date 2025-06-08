---
title: "[VM] 解決 VMWare Workstation Clone VM 後取得相同 IP"
author: Ula
tags:
  - VMWare
categories:
  - SRE
date: 2024-03-16 12:10:00
slug: vmware-clone-vm-same-ip
---

### 問題

在 VMware Workstation Pro 17 複製 VM 後開機發現每台 VM 都拿到同樣的 IP，而且改了 MAC 在重啟 VM 還是一樣。

<!--more-->

### 解決

#### 修改 MAC Address

1. 在 VMware Workstation 中，找到 cloned VM，並右鍵點擊它。
2. 在  **Edit Virtual Machine Settings**  對話方塊中，選擇  **Network Adapter**。
3. 在  **MAC Address**  欄位中，點選 Generate 重新配發新的 MAC。
4. 點擊  **OK**  儲存設定。
5. 啟動 cloned VM。

#### 修改 netplan

1. `sudo vi /etc/netplan/00-installer-config.yaml`

   ```yaml
   network:
     ethernets:
       ens33:
         dhcp4: true
         dhcp-identifier: mac # 加上這一行
     version: 2
   ```

2. `sudo netplan apply`
3. 重新查看 IP `ip addr` 便成功了

## Reference

- https://zido.site/blog/2021-07-27-ubuntu-clone-vm-same-ip/
