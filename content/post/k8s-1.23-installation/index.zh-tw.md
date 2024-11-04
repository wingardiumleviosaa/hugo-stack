---
title: 在 Rocky Linux 8 安裝 Kubernetes 1.23 (containerd as cri)
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-03-13 14:56:00
slug: install-kubernetes-123-on-rocky-linux
---
kubernetes 1.22 版之後，就不再支持 Docker 作為 container runtime 以及管理容器及鏡像的工具了。可以使用 `containerd` 取代 docker 的 container runtime；以及 `crictl` 作為 CRI(Container Runtime Interface)，另外 podman 也可以用來管理容器和鏡像。本篇記錄基於 containerd & crictl 使用 kubeadm 部屬 Kubernetes 集群的過程。

<!--more-->

## 系統環境配置 (所有節點)

### 最小系統資源需求
- 每台機器 4 GiB 以上 RAM
- master control plane 節點至少需要有兩個以上的 vCPU
- 集群中所有機器之間的完整網絡連接 (can be private or public)

Server Type  | Hostname          | Spec
:-----------:|:-----------------:|:--------------:
master     | node.ulatest.com  | 4 vCPU, 8G RAM
worker     | rockyw.ulatest.com  | 8 vCPU, 16G RAM
worker     | rockyw2.ulatest.com | 8 vCPU, 16G RAM

### 配置 /etc/hosts
```bash
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.1.5.130  nexdata nexdata.ulatest.com
10.1.5.146  node node.ulatest.com
10.1.5.147  rockyw rockyw.ulatest.com
10.1.5.148  rockyw2 rockyw2.ulatest.com

10.1.5.130  nfs nfs.ulatest.com
```
### 更新軟體套件
```
yum update -y
```

### 系統配置

#### 停用防火牆
```bash
systemctl stop firewalld
systemctl disable firewalld
```
#### 關閉 SELINUX
```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
cat /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
#### 關閉 swap
```bash
# Turn off swap
swapoff -a
# comment out the line of swap's mount point
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

#### 配置 kernel module 自動加載
```bash
cat << EOF > /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

# 執行以下命令使配置生效
modprobe overlay
modprobe br_netfilter
```

#### 調整 kernel 參數
Kubernetes 的核心是依靠 netfilter kernel module 來設定低級別的集群 IP 負載均衡，需要兩個關鍵的 module：IP轉發和橋接。
```bash
cat << EOF > /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
EOF

# 執行以下命令使配置生效
sysctl -p /etc/sysctl.d/kubernetes.conf
```
以上操作含意如下：
- 開啟 iptables 對 bridge 的數據進行處理
- 開啟數據包轉發功能（實現 vxlan）
- 禁止使用 swap 空間，只有當系統 OOM 時才允許使用它

#### 開啟 ipvs module
Kube-Proxy 是 Kubernetes 用來控制 Service 轉發過程的一個元件，預設會使用 iptables 作為 Kubernetes Service 的底層實現方式，而此模式最主要的問題是在服務多的時候產生太多的 iptables 規則，大規模情況下有明顯的性能問題。可以透過參數變化的方式要求 Kube-Proxy 使用 ipvs。開啟 ipvs 的前提條件是加載以下的 kernal module：
```bash
cat << EOF > /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack
```
上面腳本創建了的 `/etc/sysconfig/modules/ipvs.modules` 文件，保證在節點重啟後能自動加載所需模塊。使用 `lsmod | grep -e ip_vs -e nf_conntrack` 命令查看是否已經正確加載所需的內核模塊。

![](https://imgur.com/BqaYQks.png)

接下來還需要確保各個節點上已經安裝了 ipset 軟件包，以及管理工具 ipvsadm 便於查看 ipvs 的代理規則。
```bash
yum install -y ipset ipvsadm
```
如果以上前提條件如果不滿足，則即使 kube-proxy 的配置開啟了 ipvs 模式，也會退回到 iptables 模式。


## 安裝 containerd & crictl (所有節點)

### 安裝 containerd
```bash
yum install -y yum-utils
# 使用 docker.ce 作為 containerd 的 repo
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y containerd.io
systemctl enable containerd
```
生成 containerd 的配置文件:
```bash
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```
根據 [Kubernetes 文檔 Container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) 中的內容，對於使用 systemd 作為 init system 的 Linux 發行版，使用 systemd 作為容器的 cgroup driver 可以確保服務器節點在資源緊張的情況更加穩定，因此這裡配置各個節點上 containerd 的 cgroup driver 為 systemd。
- 如果檔案中`[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` 區塊下，沒有 `SystemdCgroup` 的選項，下：
```bash
sed -i 's|\(\s\+\)\[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options\]|\1\[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options\]\n\1  SystemdCgroup = true|g' /etc/containerd/config.toml
```
- 如果檔案中`[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]` 區塊下，有 `SystemdCgroup = false` 的選項，下：
```bash
sed -i 's/            SystemdCgroup = false/            SystemdCgroup = true/' /etc/containerd/config.toml
```
修改完畢後 config 內容會如下：
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```
重啟 containerd 已應用 config
```bash
systemctl restart containerd
```

### 安裝 crictl
```bash
yum install -y wget tar
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.23.0/crictl-v1.23.0-linux-amd64.tar.gz
tar zxvf crictl-v1.23.0-linux-amd64.tar.gz -C /usr/local/bin
```
設定 container runtime interface 為 containerd
```bash
cat << EOF > /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```
測試
```
crictl images
IMAGE               TAG                 IMAGE ID            SIZE
```

![](https://imgur.com/HJ9AMbi.png)

如果上方 CRI 沒有指定的話，會出現以下錯誤

![](https://imgur.com/BS6vsBx.png)


## 安裝 kubernetes 套件 (所有節點)

### 新增 kubernetes repo
```bash
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
在更新 yum 源後，使用 yum makecache 生成緩存，將套件包訊息提前在本地 cache 一份，用來提高搜索安裝套件的速度。
```bash
yum makecache -y 
```
通過 yum list 命令可以查看當前源的穩定版本，目前的穩定版本是 1.23.4-0。安裝 kubeadm 便會將 kubelet、kubectl 等依賴一併安裝。
```
yum list kubeadm
```

![](https://imgur.com/na2gxVz.png)

```
yum install -y kubeadm-1.23.4-0
```

![](https://imgur.com/q7eRhL4.png)

### 配置命令參數自動補全功能
```
yum install -y bash-completion
echo 'source <(kubectl completion bash)' >> $HOME/.bashrc
echo 'source <(kubeadm completion bash)' >> $HOME/.bashrc
source $HOME/.bashrc
```
### 啟動kubelet 服務
```
systemctl enable kubelet
systemctl restart kubelet
```

## 配置節點

### kubeadm 部署 master 節點

#### 準備配置文件
```
kubeadm config print init-defaults > kubeadm-init.yaml
vim kubeadm.yaml
```
更改以下配置
```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.1.5.146 # 改為 master node IP
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///run/containerd/containerd.sock # 改為 containerd Unix socket 地址
  imagePullPolicy: IfNotPresent
  name: rockym # 指定節點名稱
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.23.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16 # 指定 pod 子網 cidr，在設定 calico 時會用到
scheduler: {}
```
#### 叢集初始化
```
kubeadm init --config=kubeadm-init.yaml
```
完成後按照提示將 /etc/kubernetes/admin.conf 複製到 $HOME/.kube/config
```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```
並複製下面那一串加入指令以便其他 node 加入 (兩個小時過期)
```
kubeadm join 10.1.5.146:6443 --token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:fc56685aedecf323023637a6e02cc1584cfc88bfeb0690dc0e2a1feca278f008
```
或是之後使用 kubeadm token create --print-join-command 建立新的。

![](https://imgur.com/bQmARMt.png)

以上就完成 master 節點的部屬，可以使用 kubectl command 確認。

![](https://imgur.com/kW9kScR.png)

因目前網路尚未設置，所以 coredns 狀態為 Pending 是正常的。
#### 安裝 calico
```
curl -s https://docs.projectcalico.org/manifests/calico.yaml |  kubectl apply -f -
```

![](https://imgur.com/AN29VqM.png)

安裝完畢後就可以發現節點已經部屬完成了。

![](https://imgur.com/d9Is0x1.png)

### 加入工作節點
在各工作節點上直接輸入上方的 join command
```
kubeadm join 10.1.5.146:6443 --token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:fc56685aedecf323023637a6e02cc1584cfc88bfeb0690dc0e2a1feca278f008
```

![](https://imgur.com/nIV8pGu.png)

就可大功告成了~

![](https://imgur.com/Q6AICJ9.png)


{{< notice note >}}
**後記**  
有些截圖裡面可以發現原本的 master node 的 hostname 本來叫 rockym 的，可是在加入 master node 節點的時候的名字忘記改 (冏) 導致 master node 強迫改名為 node ...
求助谷歌大神，發現改節點名稱最乾淨且簡單的方式就是刪掉節點後重新加入，但不巧地是我要改的節點就是唯一一個的 master node =__= 只好折衷將錯就錯改 hostname，不知道後續會不會發生問題，先記錄一下。
{{< /notice >}}

## Reference
- [kube-proxy](https://fuckcloudnative.io/posts/ipvs-how-kubernetes-services-direct-traffic-to-pods/)
- https://kubernetes.io/zh/docs/concepts/services-networking/service/#proxy-mode-ipvs
- https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd