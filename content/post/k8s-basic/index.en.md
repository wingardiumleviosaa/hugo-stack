---
title: 'Kubernetes Introduction'
categories: ["Kubernetes"]
tags: ["Kubernetes"]
date: 2022-09-07 21:13:00
slug: kubernets-basic
---
## TL; DR
Official Definition:

{{< quote author="" source="offical site" url="https://kubernetes.io/">}}
Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications.
{{< /quote >}}

<!--more-->
Kubernetes, also known as k8s, was a project originally developed by Google using golang and then released. Used to operate automated containers, including deployment, scheduling, and scaling across node clusters.

## Architecture

![](https://imgur.com/0Innvot.png)

The picture above shows a simple Kubernetes Cluster. Usually there are multiple Masters in a Cluster as backup, but for simplicity we only show one.

## Operation Method
When the user creates a Pod through User Command (kubectl), after authenticating the user identity, the user passes the command to the API Server in the Master Node, and the API Server will back up the command to etcd. Next, the controller-manager will receive a message from the API Server that a new Pod needs to be created, and check if the resources are allowed, then create a new Pod. Finally, when the Scheduler regularly accesses the API Server, it will ask the controller-manager whether a new Pod has been created. If a newly created Pod is found, the Scheduler will be responsible for distributing the Pod to the most suitable Node.

## Nodes and Components
### Master Node
It is the console of the Kubernetes cluster, responsible for managing the cluster and coordinating all activities. It contains the following components:
#### API Server
API Server manages all API interfaces of Kubernetes and is used to communicate and operate with each node in the cluster.
#### Scheduler
Scheduler is a Pods scheduler that monitors newly created Pods that have not yet been assigned a Worker Node to run on, and coordinates the most suitable object to be placed for the Pod based on the resources on each Node.
#### Controller Manager
The component responsible for managing and running the Kubernetes controller. The controller is a number of Processes responsible for monitoring the status of the Cluster. It can be divided into the following different types:
- Node controller - Responsible for notifying and responding to the status of nodes
- Replication controller - Responsible for maintaining the set number of Pods in each replication system
- End-Point controller - Responsible for service publishing of endpoints
- Service Account & Token controller - Responsible for creating service accounts and API access tokens for newly generated Namespaces
#### etcd
It is used to store Kubernetes Cluster data as a backup. When the Master fails for some reason, we can use etcd to help us restore the state of Kubernetes.


### Worker Node
It is the runtime execution environment of Kubernetes and includes the following components:
#### Pod
Pod is the smallest unit managed by Kubernetes, which contains one or more containers and can be regarded as the logical host of an application. Containers in the same Pod share the same resources and network, and communicate with each other through local port numbers. Pod runs on a private and isolated network. By default, it is visible in other pods and services in the same cluster, but is not visible to the outside. It needs to be exposed to the outside through services.
#### Kubelet
Kubelet accepts commands from the API server to start pods and monitor status to ensure that all containers are running. It provides a heartbeat to the master node every few seconds. If the replication controller does not receive this message, the node is marked as unhealthy.
#### Kube Proxy
Perform network connection forwarding, responsible for forwarding requests to the correct container.


## Resource
- https://blog.sensu.io/how-kubernetes-works
- https://medium.com/@C.W.Hu/kubernetes-basic-concept-tutorial-e033e3504ec0
- https://ithelp.ithome.com.tw/articles/10202135