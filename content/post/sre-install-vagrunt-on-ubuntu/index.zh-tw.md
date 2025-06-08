---
title: Vagrant on Ubuntu
tags:
  - Vagrant
categories:
  - SRE
date: 2022-05-14 10:01:00
slug: install-vagrunt-on-ubuntu
---

Vagrant 是由 HashiCorp 開源、使用 Ruby 開發的虛擬機器管理工具，用於管理如 VirtualBox、VMware、AWS 等 VM，主要好處是可以提供一個可配置、可移植和復用的虛擬機環境，可快速地使用設定檔 (Vagrantfile) 和 command line 自動化安裝、配置一台 VM，降低了開發者搭建環境的時間。

<!--more-->

## Prerequisite

- Download Provider
  Vagrant 的術語中，底層的虛擬機器軟體叫作 provider，預設 provider 是 VirtualBox，其他支援的 provider 可參考[官網](https://www.vagrantup.com/docs/providers)。本文使用官方推薦的 Virtualbox，依據[官網下載步驟安裝](https://www.virtualbox.org/wiki/Downloads)。
- [Download Vagrant](https://www.vagrantup.com/downloads)

```bash
$ vagrant version
Installed Version: 2.2.19
Latest Version: 2.2.19

You're running an up-to-date version of Vagrant!
```

## Before We Start - Vagrant Basic

Vagrant 提供一個命令行工具 `vagrant`，可以直接操作虛擬機。

- Box：Vagrant 的虛擬機鏡像，可以透過在公開的 [Vagrant Box Catalog](https://app.vagrantup.com/boxes/search) 上搜尋適合的 box 使用。
- Provisioning：虛擬機實例啟動後的初始化

Vagrant 的工作流程大致如下：

- 編寫設定檔 (Vagrantfile)
- 根據設定檔下載引入 Vagrant box 檔案。
- Vagarnt 根據設定檔配置，開通並執行虛擬機器，讓它成為運行狀態。

## 建立 Vagrant 虛擬機

### 初始化

在公開的 box catalog 選定好想要的 box 後，透過 `vagrant init` 初始化，會在目錄下產生一個 Vagrantfile 檔案，建議創建一個專屬存放當下環境要用的目錄，vagrant 指令都在 Vagrantfile 所在的目錄執行，免得 Vagrant 搞錯成別台機器。

- 直接在 init 指定

```
vagrant init generic/centos7
```

![](https://imgur.com/Dgabn4i.png)

- 透過 Vagrantfile 指定
  若不加 box 名稱，可直接下 `vagrant init`，再去修改 Vagrantfile 中的相關參數。

![](https://imgur.com/aqfMYZp.png)

![](https://imgur.com/Ox8nOm3.png)

### 啟動

啟動虛擬機，第一次啟動須下載整份 box 的檔案，故可能會花幾分鐘的時間來啟動。

```
vagrant up
```

![](https://imgur.com/17sva0x.png)

確認當前 Vagrant 主機的運作狀況

```
vagrant status
```

![](https://imgur.com/BStuf6X.png)
在啟動完成後，透過 SSH 來登入該虛擬機。

```
vagrant ssh
```

![](https://imgur.com/F6k1taT.png)

打開 Virtualbox 可以看到 VM 透過 command line 迅速的建立完成了，而不用透過原始的方式從 iso 開始從頭安裝。

![](https://imgur.com/lhECdc7.png)

## 其他 vagrant 虛擬機的操作

```ruby
# 將虛擬機關機，會先嘗試優雅關機 (gracefully shutdown)，若失敗了或者指令有加上 -f 旗標，就會直接將虛擬機電源關閉。
vagrant halt

# 重新開機，當用於修改 Vagrantfile 後，使之生效。
vagrant reload

# 徹底移除虛擬機器
vagrant destroy

# 暫停機器，會保留暫停時的狀態，可在下次重新快速啟動，但會需要額外的空間來放記錄檔，且會佔用 RAM
vagrant suspend

# 恢復機器
vagrant resume

# 把當前的運行的虛擬機環境進行打包為 box 文件
vagrant package
```

## vagrant box 操作

可用來管理 box，有以下指令

```ruby
# 手動下載至本機
vagrant box add <box_name> <box_url>

# 列出本機已下載的 box
vagrant box list

# 刪除指定 box
vagrant box remove <box_name>

# 檢查 box 是否有新的版本
vagrant box outdated --box <box_name>

# 升級 box
vagrant box update --box <box_name>

# 移除舊版 box
vagrant box prune --box <box_name>
```

## Vagrantfile 配置檔說明

Vagrantfile 就是每一台的虛擬機的規格表，使用 ruby 語法撰寫，看起來很長，但其實幾乎都是註解說明。

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "generic/centos7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"
```

## 基本設置

- config.vm.box：指定連接的 box 名稱
- config.vm.hostname：指定主機名稱
- config.vm.define：命名虛擬機，若沒指定則預設為 default

### 網路配置

有幾種不同的配置

- 公有網絡（bridge)
  虛擬機和宿主機相當於局域網中獨立的主機，設置靜態 IP：

```ruby
config.vm.network "public_network", ip: "192.168.1.120"
```

如果使用 public_network 而不配置 IP，那麼會 DHCP 自動獲取 IP 地址。

- 私有網絡（host-only)
  只有宿主機能訪問虛擬機，多個虛擬機在同一個網段，相互可以訪問：

```ruby
config.vm.network "private_network", ip: "192.168.21.4"
```

- 端口映射
  將宿主機端口映射到虛擬機端口，例如宿主機 8080 端口映射到虛擬機 80 端口：

```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
```

### 共享目錄 Synced Folders

vagrant 啟動時會預設將 Vagrantfile 位置掛載到虛擬機中的 `/vagrant`，如果要掛載其他目錄的話使用：

```ruby
config.vm.synced_folder "src/", "/srv/website"
```

將本機的 src/ 目錄(相對路徑)掛載到虛擬機內的 /srv/website 中。

### Provisioning

如果要一次執行多的相同環境的 VM，配置所需要的環境每次都要 ssh 個別進去裝太慢了，Vagrant 可以利用先寫好的自動安裝環境腳本來達到自動化配置。支持 Shell，Puppet，Chef，Ansible 等等：

- shell

```ruby
config.vm.provision "shell", run: "always", inline: <<-SHELL
    sudo yum install -y net-tools
SHELL
```

run: “always” 表示每次 vagrant up 的時候，都執行 Provision。

- 使用外部 shell script 腳本

```ruby
config.vm.provision "shell", path: "script.sh
```

- Ansible

```ruby
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
```

- 使用虛擬機內部的 Ansible：

```ruby
Vagrant.configure("2") do |config|
  # Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
  end
end
```

並非每次 vagrant up 的時候，都會執行 Provision。只有在下面 3 種情況下 Provision 才會執行：

- 首次執行 vagrant up
- 執行 vagrant provision
- 執行 vagrant reload --provision

### Provider

可以另外自己設置 provider 的參數，以 virtualbox 為例，VirtualBox 提供了 VBoxManage 這個命令列工具，可以讓我們設定 VM，用 modifyvm 這個命令讓我們可以設定 VM 的名稱和記憶體大小等等，詳細可以設定的參數可以參考 [virtualbox 官網的介紹](https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm)。

```ruby
Vagrant.configure("2") do |config|
  # ...
  config.vm.provider "virtualbox" do |vb|
    # 設定 CPU 使用率最多只能是本機的 50%
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
  end
end
```
