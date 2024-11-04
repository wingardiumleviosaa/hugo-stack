---
title: Calico Running but Unready (Ready 0/1)
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-05-19 08:53:00
slug: calico-running-but-unready
---
## 環境說明
- Kubernetes 1.20.10
- Calico 3.23

<!--more-->

## 錯誤描述
部署 calico 網絡後狀態雖為 Running，但 container 都無法成功運作，calico-controller 以及放在每個節點的 calico-node 皆無法初始化，如下圖

![](https://imgur.com/gfqwGhG.png)

- calico-kube-controllers 日誌
```bash
2022-05-17 01:11:09.503 [FATAL][1] main.go 120: Failed to initialize Calico datastore error=Get "https://10.254.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default": context deadline exceeded
2022-05-17 01:11:09.962 [INFO][1] main.go 94: Loaded configuration from environment config=&config.Config{LogLevel:"info", WorkloadEndpointWorkers:1, ProfileWorkers:1, PolicyWorkers:1, NodeWorkers:1, Kubeconfig:"", DatastoreType:"kubernetes"}
W0822 01:11:09.963646 1 client_config.go:615] Neither --kubeconfig nor --master was specified. Using the inClusterConfig. This might not work.
2022-05-17 01:11:09.964 [INFO][1] main.go 115: Ensuring Calico datastore is initialized
```
- calico-node 日誌
```bash
2022-05-17 01:11:14.102 [WARNING][69] felix/health.go 211: Reporter is not ready. name="async_calc_graph"
2022-05-17 01:11:14.102 [WARNING][69] felix/health.go 211: Reporter is not ready. name="int_dataplane"
2022-05-17 01:11:14.103 [WARNING][69] felix/health.go 173: Health: not ready
2022-05-17 01:11:14.540 [INFO][69] felix/watchercache.go 180: Full resync is required ListRoot="/calico/resources/v3/projectcalico.org/kubernetesendpointslices"
2022-05-17 01:11:14.544 [INFO][69] felix/watchercache.go 193: Failed to perform list of current data during resync ListRoot="/calico/resources/v3/projectcalico.org/kubernetesendpointslices" error=resource does not exist: KubernetesEndpointSlice with error: the server could not find the requested resource
```

## 原因及解法
原因是因為安裝的 Calico 版本不相容於 Kubernetes，依據 [Calico 官網](https://projectcalico.docs.tigera.io/archive/v3.23/getting-started/kubernetes/requirements) 的描述，3.23 版最低支援的 k8s 版本為 1.21，故發生該錯誤。
解決方法是將 Calico 重新部署為 3.21 版，以相容於 k8s 1.20 的環境
```bash
curl -s https://docs.projectcalico.org/manifests/calico.yaml |  kubectl delete -f -
  configmap "calico-config" deleted
  customresourcedefinition.apiextensions.k8s.io "bgpconfigurations.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "bgppeers.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "blockaffinities.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "caliconodestatuses.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "clusterinformations.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "felixconfigurations.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "globalnetworkpolicies.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "globalnetworksets.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "hostendpoints.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "ipamblocks.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "ipamconfigs.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "ipamhandles.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "ippools.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "ipreservations.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "kubecontrollersconfigurations.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "networkpolicies.crd.projectcalico.org" deleted
  customresourcedefinition.apiextensions.k8s.io "networksets.crd.projectcalico.org" deleted
  clusterrole.rbac.authorization.k8s.io "calico-kube-controllers" deleted
  clusterrolebinding.rbac.authorization.k8s.io "calico-kube-controllers" deleted
  clusterrole.rbac.authorization.k8s.io "calico-node" deleted
  clusterrolebinding.rbac.authorization.k8s.io "calico-node" deleted
  daemonset.apps "calico-node" deleted
  serviceaccount "calico-node" deleted
  deployment.apps "calico-kube-controllers" deleted
  serviceaccount "calico-kube-controllers" deleted
  error: unable to recognize "STDIN": no matches for kind "PodDisruptionBudget" in version "policy/v1"
```
重新部署
```
curl -s https://docs.projectcalico.org/v3.21/manifests/calico.yaml | kubectl apply -f -
  configmap/calico-config created
  Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
  customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
  customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
  clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
  clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
  clusterrole.rbac.authorization.k8s.io/calico-node created
  clusterrolebinding.rbac.authorization.k8s.io/calico-node created
  daemonset.apps/calico-node created
  serviceaccount/calico-node created
  deployment.apps/calico-kube-controllers created
  serviceaccount/calico-kube-controllers created
```
部署依照環境配置大概需要等幾分鐘不等，過程中可能會造成其他 Pod 被重啟，觀察下來是正常的情況

![](https://imgur.com/RhKQ9ei.png)

全部完成後就可以看到 pod 皆正常運作了

![](https://imgur.com/IHq4Sx8.png)


## Reference
- https://projectcalico.docs.tigera.io/archive/v3.21/getting-started/kubernetes/requirements
- https://github.com/opsnull/follow-me-install-kubernetes-cluster/issues/633