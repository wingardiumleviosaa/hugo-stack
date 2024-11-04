---
title: 'Kubernetes Object 基本對象介紹'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2021-01-07 17:38:00
slug: kubernets-object
---
## K8s Obejcts

### 常用的基本 objects

- Pod  
Pod 有兩種類型：普通 Pod 和靜態 Pod (static pod)。靜態 Pod 即不通過 K8S 調度和創建，直接在某個具體的 Node 機器上通過具體的文件來啟動。普通 Pod 則是由 K8S 創建、調度，同時數據存放在 etcd 中。
<!--more-->
- Service  
可以認為是 pod 的反向代理，負責接收客戶端請求，把請求轉給 pod。因為每個 pod 都有自己的內部 ip，但是 deployment 的 pod 的 ip 是有可能變的 (pod 掛掉或複製)，所以需要 service 來做類似中間者的抽象存在。Service 挑選、關聯 Pod 的方式為基於 Label Selector 進行定義。通過 `type` 在 `ServiceSpec`中指定可以以不同的方式公開服務：
    - ClusterIP (default)：只有內部 IP，只能從集群內訪問服務。
    - NodePort：工作於每個節點的主機 IP 之上，可以從集群外部訪問服務。ClusterIP 的超集。
    - LoadBalancer：在當前集群中創建一個外部負載平衡器，把外部請求負載均衡至多個Node 主機 IP 的 NodePort 之上，為該服務分配一個固定的外部 IP。NodePort 的超集。
    - ExternalName-externalName：通過在 Cluster 的 DNS Server 添加一筆 CName Record，使用指定名稱（在 yaml 中設定）公開服務，不使用代理 (kube-proxy)，而是透過 kube-dns。此類型在 kubernetes 1.7 版本有提供，但是要 kube-dns version 要在 1.14.9 以上，否則會遇到 Resolve External Name issue。主要是為了讓不同 namespace 中的 Service 可以利用 ExternalName 設定的外部名稱連到其它的 namespace 中的 Service。
- Label  
標籤用於區分對象，使用標籤引用對象而不再是 IP。Label 以鍵值對的形式存在，每個對象可以有多個標籤，通過標籤可以關聯對象。
- Volume  
共享 Pod 中使用的數據。
- Namespace  
可以抽象理解為一群對象的集合。

### High-level objects (Controllers)
建立在基本對象的基礎上，提供了附加的功能和便利性：
- ReplicaSet：確保運行指定數量的 pod，官方建議使用 Deployment 來自動管理。
- Deployment：最常見的部屬 Pod 的方法，可以用來創建 replicaSet。支持版本記錄、rolling update/back、暫停升級等高級特性。
- StatefulSet：用於管理具有持久性存儲的有狀態應用程序。
    - 有序部屬、有序擴展：即 Pod 是有順序的，在部署或者擴展的時候要依據定義的順序依次依次進行（即從 0 到 N-1 ，在下一個 Pod 運行之前所有之前的 Pod 必須都是 Running 和 Ready 狀態），基於 init containers 來實現。所以 pod name 固定且會帶一個有序的序號(app-0, app-1...)。
    - 持久性儲存：基於 pvc 實現，pod 重新調度後仍會跟 volume 保持關聯， volume 不會隨 pod 刪除而刪除。
    - 穩定網路標誌：基於 Headless Service（即沒有 Cluster IP 的 Service）來實現，Pod 重新調度後其 PodName 和 HostName 不變。
- DaemonSet：保證在每個 Node 上都運行一個容器副本，常用來部署一些集群的日誌、監控或者其他系統管理應用。
- Job：一次性任務，運行完後Pod銷毀，不再自動重建。
- CronJob：定時任務。


## Resource
- https://www.cnblogs.com/ccbloom/p/11311286.html#%E5%9F%BA%E7%A1%80%E5%AF%B9%E8%B1%A1
- https://www.cnblogs.com/baoshu/p/13124881.html