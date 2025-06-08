---
title: "[KVM] virt-install 建立 CentOS 虛擬機"
tags:
  - KVM
categories:
  - SRE
date: 2021-09-16 10:52:00
slug: kvm-virt-install-centos-vm
---

## 安裝 KVM 套件

檢查 CPU 是否支援虛擬化

<!--more-->

```bash
lscpu | grep -i virtualization
```

![](https://imgur.com/KH80Y69.png)

加入 qemu repo

```bash
yum install -y centos-release-qemu-ev
```

安裝套件

```bash
yum install -y qemu-kvm-ev libvirt libvirt-python libguestfs-tools virt-install
```

確認 kvm module 有正常載入，如果執行結果有 kvm_xxx(CPU 系列）則說明 kvm 服務已經啟動

```bash
lsmod | grep kvm
```

![](https://imgur.com/BNCFa7v.png)

如果未啟動，可通過如下命令啟動

```bash
systemctl start libvirtd
```

將服務設為開機啟用

```bash
systemctl enable libvirtd
systemctl is-enabled libvirtd
```

## 配置 KVM 網絡橋接功能

產生虛擬橋接器，並將實體網卡納入該 bridge 之下，使得將來 VM 可以使用 bridge 模式，具有獨立對外 IP。

![](https://imgur.com/x72LCiP.png)

本機網路設定

```bash
nmcli con add type bridge ifname br0
nmcli connection show

# 10.1.5.1 為實體網卡個網路位置
nmcli con modify bridge-br0 ipv4.method manual ipv4.address "10.1.5.1/24" ipv4.gateway "10.1.5.254" ipv4.dns "10.1.1.3" autoconnect yes

# 把原先使用的網卡 ens1f0 加在 bridge 裡
nmcli con add type bridge-slave ifname ens1f0 master bridge-br0

# 啟用 bridge
nmcli connection up bridge-br0

# 重啟網路
systemctl restart network
nmcli connection show
```

KVM 網路設定

```bash
# 其中 uuid 的欄位可用 uuidgen 產生
cat << EOF > br0.xml
<network>
  <name>br0</name>
  <uuid>f9eae5e5-6bdd-4c72-bd10-6b97b6452126</uuid>
  <bridge name="br0"/>
  <forward mode='bridge'/>
</network>
EOF

# 新增網卡並啟用
virsh net-define br0.xml
virsh net-start br0
virsh net-autostart br0

# 將原先的 default 網卡刪掉
virsh net-undefine default
virsh net-destroy default
virsh net-list --all
```

## 開始創建 (從 iso 建立)

```bash
$ virt-install --name centos7.9 \
--disk path=/var/lib/libvirt/images/centos7.9,size=90,format=qcow2 \
--vcpus 4 --memory 4096 --network bridge=br0 \
--graphics none --os-type linux --os-variant centos7.0 \
--location /var/lib/libvirt/images/CentOS-7-x86_64-Minimal-2009.iso \
--extra-args 'console=ttyS0'
```

參數說明

- `--disk` 指定該虛擬機使用的存儲，有多種介質可選。比如指定本地文件可以使用 path 選項，如果指定文件不存在還需要設置 size 參數。如果不配置磁盤可以使用 `--disk none`
- `--graphics` 可選項，指定 guest 圖像顯示配置，如果不需要圖像顯示，可以使用 `--graphics none`
- `--console` 可選項，virt-install 會默認配置合適的 console，可以不填。
- `--os-type` 是指定作業系統類型（可為 `linux` 或 `windows`）
- `--os-variant` 可選項，指定 guest 操作系統，用於優化配置。可用的選項可以使用 `osinfo-query` 指令查詢

  ```
  osinfo-query os | grep CentOS
  ```

- 安裝源參數
  - `--location` 指定新創建虛擬機的安裝介質。使用該參數指定安裝介質時，默認是看不到 guest 安裝過程中的輸出文本的，需要另外配置參數`--extra-args 'console=ttyS0'`
  - `--import` 指定新創建虛擬機跳過安裝階段，使用第一個 `--disk`
- `--network` 網路

  指定虛擬機連接的網絡配置，常用的兩種：

  - `bridge=BRIDGE`
    指定連接到 host 上名為 BRIDGE 的虛擬網橋上。
  - `network=NAME`
    指定連接到 virsh 管控的名為 NAME 的 network 上。

下完 virt-install 後就會跳出 CentOS 的命令列安裝配置畫面，有 `[!]` 基本都是要配置，按照順序往下配置，按對應的數字以進行設定。

```
Installation

 1) [x] Language settings                 2) [!] Timezone settings
        (English (United States))                (Timezone is not set.)
 3) [!] Installation source               4) [!] Software selection
        (Processing...)                          (Processing...)
 5) [!] Installation Destination          6) [x] Kdump
        (No disks selected)                      (Kdump is enabled)
 7) [ ] Network configuration             8) [!] Root password
        (Not connected)                          (Password is not set.)
 9) [!] User creation
        (No user will be created)
  Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]:
```

- 3 的安裝源請選擇 `2) local ISO file`
- 5 Installation Destination 選擇安裝目的地並選擇 `2) Use All Space 使用所有空間`；最後分割方案選擇 `3) LVM 邏輯卷管理`

## 開始創建 (從已存在的 qcow2 image 建立)

```bash
virt-install --name centos \
--vcpus 4 --memory 4096 \
--disk /var/lib/libvirt/images/centos \
--network bridge=br0 --boot hd
```

## 完成安裝後進入 VM 設置網路

通過 virsh console <虛擬機器名稱> 命令來連線虛擬機器

```
# 檢視虛擬機器
$ virsh list              # 檢視在執行的虛擬機器
$ virsh list --all         # 檢視所有虛擬機器

 Id    Name                           State
----------------------------------------------------
 7     centos7.9                       running
```

連線虛擬機器

```
$ virsh console centos7.9
```

配置虛擬機器網路，編輯 `vi /etc/sysconfig/network-scripts/ifcfg-eth0`

```
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eth0
UUID=adfa3b7d-bf60-47e6-8482-871dee686fb5
DEVICE=eth0
ONBOOT=yes
IPADDR=10.1.5.1
PREFIX=24
GATEWAY=10.1.5.254
DNS1=10.1.1.3
```

重啟網路

```
$ systemctl restart network
```

## 虛擬機的其他管理命令

```
virsh start centos7.9     # 虛擬機器開啟（啟動）：
virsh reboot centos7.9    # 虛擬機器重新啟動
virsh shutdown centos7.9  # 虛擬機器關機
virsh destroy centos7.9   # 強制關機（強制斷電）
virsh suspend centos7.9   # 暫停（掛起）KVM 虛擬機器
virsh resume centos7.9    # 恢復被掛起的 KVM 虛擬機器
virsh undefine centos7.9  # 該方法只刪除配置檔案，磁碟檔案未刪除
virsh autostart centos7.9 # 隨物理機啟動而啟動（開機啟動）
virsh autostart --disable centos7.9 # 取消標記為自動開始（取消開機啟動）
```

## clone VM

```
[root@ula ~]# virt-clone --connect qemu:///system -o rocky --name rocky2 -f /var/lib/libvirt/images/rocky2.qcow2
Allocating 'rocky2.qcow2'

Clone 'rocky2' created successfully.
[root@ula ~]#
```

- `--connect=URI`：連接到虛擬機管理程序 libvirt 的 URI
- `-o ORIGINAL_GUEST` / `--original=ORIGINAL_GUEST`：原虛擬機名稱，原 VM 必須處於關機狀態
- `-n NEW_NAME` / `--name=NEW_NAME`：新的虛擬機名稱
- `--auto-clone`：從原來的虛擬機配置自動生成克隆名稱和存儲路徑。例如，原虛擬機名為 VM01，那克隆的虛擬機名為 VM01-clone；原虛擬機的磁盤路徑為 /usr/src/VM01.img，克隆後的虛擬機磁盤路徑為 /usr/src/VM01-clone.img
- `-f NEW_DISKFILE` / `--file=NEW_DISKFILE`：指定新的虛擬機磁盤文件

## Reference

- 關於更多 kvm 網路模式可以參考這篇分享 https://www.samyang.top/2018/12/kvm/
- https://www.itread01.com/content/1545319657.html
