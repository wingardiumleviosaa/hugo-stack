---
title: Install KVM (libvert) by Vagrant on CentOS
tags:
  - Vagrant
  - KVM
categories:
  - SRE
date: 2022-05-16 17:49:00
slug: install-kvm-by-vagrant-on-centos
---

上次紀錄了如何在 Ubuntu 透過 Virtualbox 使用 Vagrant，本篇文章記錄如何在 CentOS 透過 KVM 使用 Vagrant 自動化建立 VM。

<!--more-->

## Prerequisite

- 安裝 KVM，請參考之前[文章記錄](https://wingardiumleviosaa.github.io/KVM-virt-install-CentOS/)
- 安裝 [Vagrant](https://www.vagrantup.com/downloads)

## 安裝 Plugin

- vagrant-libvirt
  用於支持 libvirt
  ```
  yum install -y libvirt-devel
  vagrant plugin install vagrant-libvirt
  ```
  需要安裝好 libvirt-devel 才能安裝 vagrant-libvirt 插件，否則提示以下錯誤
  ```
  libvirt library not found in default locations (RuntimeError)
  ```
- vagrant-mutate
  用於將官方的 Vagrant guest box 轉換成 KVM 格式
  ```
  vagrant plugin install vagrant-mutate
  ```
  在 Vagrant 官方鏡像中，也有部分發行版直接提供了 libvirt 版本

## 定義 Vagrantfile

```ruby
ENV["LC_ALL"] = "en_US.UTF-8"

MASTER=3
WORKER=2

Vagrant.configure("2") do |config|

$script_ansible = <<EOF
    yum install -y net-tools git centos-release-ansible-29
    yum install -y ansible
    localectl set-locale LANG=en_US.UTF-8
    source /etc/locale.conf
    mkdir -p /root/.ssh
    cp /home/vagrant/.ssh/authorized_keys /root/.ssh/
    cp /home/vagrant/.ssh/id_rsa /root/.ssh/
EOF

$script = <<EOF
    yum install -y net-tools
    localectl set-locale LANG=zh_TW.UTF-8
    source /etc/locale.conf
    mkdir -p /root/.ssh
    cp /home/vagrant/.ssh/authorized_keys /root/.ssh/
EOF

  config.ssh.insert_key = false
  config.vm.provider "libvirt" do |v|
    v.memory = 8192
    v.cpus = 4
    v.storage_pool_name = "images"
  end

  (1..MASTER).each do |i|
    config.vm.define "master#{i}" do |node|
      node.vm.box = "generic/centos7"
      node.vm.hostname = "master#{i}"
      # node.vm.network :private_network, ip: "192.168.56.2#{i}", :netmask => "255.255.255.0"
      node.vm.network "public_network"
           :dev=>"br0"
      node.vm.synced_folder "/root/k8sInstaller", "/etc/ansible/roles/k8sInstaller"
      node.vm.provision "file", source: "~/.vagrant.d/insecure_private_key", destination: "~/.ssh/id_rsa"
      node.vm.provision :shell, :inline => $script_ansible
    end
  end

(1..WORKER).each do |i|
    config.vm.define "worker#{i}" do |node|
      node.vm.box = "generic/centos7"
      node.vm.hostname = "worker#{i}"
      # node.vm.network :private_network, ip: "192.168.56.3#{i}", :netmask => "255.255.255.0"
      node.vm.network "public_network"
           :dev=>"br0"
      node.vm.provision :shell, :inline => $script
    end
  end

end
```

更多 provider 設定可參考 [vagrant-libvert](https://github.com/vagrant-libvirt/vagrant-libvirt#vagrant-libvirt-provider) 文檔。

## 啟動

執行 vagrant up 指令需要額外指定 provider 如下：

```
vgrant up --provider=libvirt
```

或者在執行前設定 VAGRANT_DEFAULT_PROVIDER 環境變數：

```
export VAGRANT_DEFAULT_PROVIDER=libvirt
```

成功的話，可以看到虛擬機列表有五台用 vagrant 啟動的機器

![](https://imgur.com/3vGJKRr.png)

## Call to virStoragePoolDefineXML failed: operation failed: Storage source conflict with pool: 'images'

CentOS 7 部署 kvm 使用 SELinux 增強模式，不能使用非默認的目錄來創建 VM 鏡像。Vagrant 會嘗試使用一個名為 default 的存儲池，如果不存在就會嘗試在 /var/lib/libvirt/images 上創建 defualt 存儲池。返回失敗的原因是已經在此為址創建了 images 儲存池。

```
virsh pool-list --all
Name                 State      Autostart
-------------------------------------------
 images               active     yes
```

直接定義 Vagrant 的存儲池使用 images，在 Libvirt 配置中，有關 Porvider Options 中可以使用 storage_pool_name 設置 Libvirt 存儲池名字，也就是 box image 和 instance snapshoot 存儲位置。
在 Vagrantfile 中添加配置

```ruby
Vagrant.configure("2") do |config|
  ...
  config.vm.provider :libvirt do |vm|
    vm.storage_pool_name = "images"
  end
  ...
end
```

然後再次執行命令 `vagrant up --provider libvirt` 就可以成功安裝。

## Call to virDomainCreateWithFlags failed: Unable to get index for interface eth0: No such device

如果網路使用 public network，Vagrant 預設使用來當 bridge 的網卡是 eth0，而發生該錯誤訊息代表本地機器中找不到 eth0 網卡，請在設定檔中加上 dev 指定網卡名稱。

```ruby
Vagrant.configure("2") do |config|
  ...
  config.vm.network "public_network"
        :dev => "br0"
  ...
end
```

其他參數可以參考 [public network options](https://github.com/vagrant-libvirt/vagrant-libvirt#public-network-options)

## Reference

- https://huataihuang.gitbooks.io/cloud-atlas/content/virtual/vagrant/vagrant_libvirt_kvm.html
