---
title: >-
  Unable to connect to the server: x509: certificate has expired or
  is not yet valid
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2021-11-21 17:36:00
slug: kubernets-ca-expired
---
在下 `kubectl` 時出現 `Unable to connect to the server: x509: certificate has expired or is not yet valid` 的錯誤，原因是 kubernetes apiserver 證書已過期，kubernetes 的 apiServer 與 kubelet 的訪問授權證書是一年，官方表示通過這種方式，讓用戶不斷的升級版本。

<!--more-->

目前有幾種解決方式：

- 重新生成證書取代過期的證書 (本次作法)
- 升級集群以自動更新證書
- 部屬一套新的環境，將業務遷移過去
- 去掉證書驗證功能 (不安全且不科學，需要自己改 source code)


## 查看證書的有效日期

透過 `openssl` 直接查證書內容

```bash
$ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text | grep Not
            Not Before: Nov 17 04:48:20 2020 GMT
            Not After : Nov 17 04:48:20 2021 GMT
```

或是透過 `kubeadm` 檢查 Kubernetes 環境證書

```bash
$ kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[check-expiration] Error reading configuration from the Cluster. Falling back to default configuration

W1118 09:51:35.880390    7092 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Nov 17, 2021 04:48 UTC   <invalid>                               no
apiserver                  Nov 17, 2021 04:48 UTC   <invalid>       ca                      no
apiserver-etcd-client      Nov 17, 2021 04:48 UTC   <invalid>       etcd-ca                 no
apiserver-kubelet-client   Nov 17, 2021 04:48 UTC   <invalid>       ca                      no
controller-manager.conf    Nov 17, 2021 04:48 UTC   <invalid>                               no
etcd-healthcheck-client    Nov 17, 2021 04:48 UTC   <invalid>       etcd-ca                 no
etcd-peer                  Nov 17, 2021 04:48 UTC   <invalid>       etcd-ca                 no
etcd-server                Nov 17, 2021 04:48 UTC   <invalid>       etcd-ca                 no
front-proxy-client         Nov 17, 2021 04:48 UTC   <invalid>       front-proxy-ca          no
scheduler.conf             Nov 17, 2021 04:48 UTC   <invalid>                               no

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Nov 15, 2030 04:48 UTC   8y              no
etcd-ca                 Nov 15, 2030 04:48 UTC   8y              no
front-proxy-ca          Nov 15, 2030 04:48 UTC   8y              no
```

經查看 k8s master 組件證書都過期了。

## 更新證書

1. 備份舊有的配置文件與證書

```bash
$ cp -rf /etc/kubernetes /etc/kubernets.bak
```

2. 更新證書

```bash
$ kubeadm alpha certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[renew] Error reading configuration from the Cluster. Falling back to default configuration

W1118 11:11:52.322016   26585 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
```

3. 重新生成配置文件

	這些配置文件中包含證書，所以需要重新生成

```bash
$ rm -rf /etc/kubernetes/*.conf
$ kubeadm init phase kubeconfig all --apiserver-advertise-address 10.1.5.21
```

4. 更新配置身份認證的 `$HOME/.kube/config` 檔案  
	將重新生成於 `/etc/kubernetes` 下的 `admin.conf` 檔案覆蓋原先的 `~/.kube/config`

```bash
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ chown $(id -u):$(id -g) $HOME/.kube/config
```

5. 重新啟動 kubelet & docker service

```bash
$ systemctl restart kubelet
$ systemctl restart docker
```

6. 重新使用 kubectl 訪問集群

```bash
$ kubectl get nodes
NAME    STATUS   ROLES    AGE    VERSION
k8sm1   Ready    master   365d   v1.17.13
k8sm2   Ready    master   365d   v1.17.13
k8sm3   Ready    master   365d   v1.17.13
k8sw1   Ready    <none>   365d   v1.17.13
k8sw2   Ready    <none>   365d   v1.17.13
k8sw3   Ready    <none>   365d   v1.17.13
```

7. 如果是多 master，上面的步驟在每個 master 都要做



## Reference
- https://stackoverflow.com/questions/56320930/renew-kubernetes-pki-after-expired
- https://cloud.tencent.com/developer/article/1832411