---
title: 'Kubernetes Introduction'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2022-09-07 21:13:00
slug: kubernets-basic
---
## 簡介
官方定義:

{{< quote author="" source="offical site" url="https://kubernetes.io/">}}
Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.
{{< /quote >}}

<!--more-->
Kubernetes 又稱為 k8s，最初由 google 用 golang 開發而後釋出的專案。用於操作自動化容器，包括部署，調度和節點集群間擴展。

## 架構

![](https://imgur.com/0Innvot.png)

上圖為一個簡易的 Kubernetes Cluster，通常一個 Cluster 中會有多個 Master 作為備援，但為了簡化我們只顯示一個。

## 運作方式
使用者透過 User Command（kubectl）建立 Pod 時經過使用者身份的認證後，再將指令傳遞到 Master Node 中的 API Server，API Server 會把指令備份到 etcd 。接下來 controller-manager 會從 API Server 收到需要創建一個新的 Pod 的訊息，並檢查如果資源許可，就會建立一個新的 Pod。最後 Scheduler 在定期訪問 API Server 時，會詢問 controller-manager 是否有建置新的 Pod，如果發現新建立的 Pod 時，Scheduler 就會負責把 Pod 配送到最適合的一個 Node 上面。

## 節點與組件
### Master Node
為 Kubernetes 叢集的控制台，負責管理集群、協調所有活動，包含的元件如下：
#### API Server
API Server 管理 Kubernetes 的所有 api interface，用來和集群中的各節點通訊並進行操作。
#### Scheduler
Scheduler 是 Pods 調度員，監視新建立但還沒有被指定要跑在哪個 Worker Node 上的 Pod，並根據每個 Node 上面資源去協調出一個最適合放置的對象給該 Pod。
#### Controller Manager
負責管理並運行 Kubernetes controller 的組件，controller 是許多負責監視 Cluster 狀態的 Process，又可分為下列不同的種類
- Node controller - 負責通知與回應節點的狀態
- Replication controller - 負責每個複寫系統內維持設定的 Pod 數量
- End-Point controller - 負責端點的服務發布
- Service Account & Token controller - 負責創建服務帳戶與新生成的 Namespace 的 API 存取 Token
#### etcd
用來存放 Kubernetes Cluster 的資料作為備份，當 Master 因為某些原因而故障時，我們可以透過 etcd 幫我們還原 Kubernetes 的狀態。


### Worker Node
為 Kubernetes 的 runtime 執行環境，包含的元件如下：
#### Pod
Kubernetes pod 是 Kubernetes 管理的最小單元，裡面包含一個或多個 container，可視為一個應用程式的邏輯主機。 同一個 Pod 中的 Containers 共享相同資源及網路，彼此透過 local port number 溝通。pod 運行在私有隔離的網絡上，默認情況下在同一集群的其他 pod 和 service 中可見，但是外部不可見，需要藉助 service 暴露給外部。
#### Kubelet
Kubelet 接受 API server 的命令，用來啟動 pod 並監測狀態，確保所有 container 都在運行。它每隔幾秒鐘向 master node 提供一次 heartbeat。如果 replication controller 未收到該消息，則將該節點標記為不正常。
#### Kube Proxy
進行網路連線的 forwarding，負責將 request 轉發到正確的 container。



## Resource
- https://blog.sensu.io/how-kubernetes-works
- https://medium.com/@C.W.Hu/kubernetes-basic-concept-tutorial-e033e3504ec0
- https://ithelp.ithome.com.tw/articles/10202135