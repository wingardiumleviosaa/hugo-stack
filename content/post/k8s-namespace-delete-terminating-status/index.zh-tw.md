---
title: Kubernetes namespace 一直 delete 不成功的原因 (卡在 terminating status)
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-03-22 14:49:00
slug: kubernetes-namespace-delete-terminating
---

最近在刪除 namespace 的時候總是會卡在 Terminating 的狀態，一直不疑有他的直接使用網路上常看的解決方法將 spec.finalizers 清空。但因為每次刪、每次卡，就連完全無任何資源的命名空間也是卡！仔細看後才發現原來是有其他元件錯誤，進而造成影響。

<!--more-->

## 原因排查
先使用清空 finalizer 的方法強制刪除 namespace，
```
kubectl proxy
Starting to serve on 127.0.0.1:8001
```
另開一個 terminal
```
cat <<EOF | curl -X PUT \
  localhost:8001/api/v1/namespaces/test/finalize \
  -H "Content-Type: application/json" \
  --data-binary @-
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "test"
  },
  "spec": {
    "finalizers": null
  }
}
EOF
```
會回傳下面結果
```yaml
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "test",
    "uid": "353766ea-be97-4ccf-9275-0b39bd651afe",
    "resourceVersion": "1359585",
    "creationTimestamp": "2022-03-21T04:18:38Z",
    "deletionTimestamp": "2022-03-21T04:18:51Z",
    "labels": {
      "kubernetes.io/metadata.name": "test"
    },
    "managedFields": [
      {
        "manager": "kubectl-create",
        "operation": "Update",
        "apiVersion": "v1",
        "time": "2022-03-21T04:18:38Z",
        "fieldsType": "FieldsV1",
        "fieldsV1": {
          "f:metadata": {
            "f:labels": {
              ".": {},
              "f:kubernetes.io/metadata.name": {}
            }
          }
        }
      },
      {
        "manager": "kube-controller-manager",
        "operation": "Update",
        "apiVersion": "v1",
        "time": "2022-03-21T04:18:57Z",
        "fieldsType": "FieldsV1",
        "fieldsV1": {
          "f:status": {
            "f:conditions": {
              ".": {},
              "k:{\"type\":\"NamespaceContentRemaining\"}": {
                ".": {},
                "f:lastTransitionTime": {},
                "f:message": {},
                "f:reason": {},
                "f:status": {},
                "f:type": {}
              },
              "k:{\"type\":\"NamespaceDeletionContentFailure\"}": {
                ".": {},
                "f:lastTransitionTime": {},
                "f:message": {},
                "f:reason": {},
                "f:status": {},
                "f:type": {}
              },
              "k:{\"type\":\"NamespaceDeletionDiscoveryFailure\"}": {
                ".": {},
                "f:lastTransitionTime": {},
                "f:message": {},
                "f:reason": {},
                "f:status": {},
                "f:type": {}
              },
              "k:{\"type\":\"NamespaceDeletionGroupVersionParsingFailure\"}": {
                ".": {},
                "f:lastTransitionTime": {},
                "f:message": {},
                "f:reason": {},
                "f:status": {},
                "f:type": {}
              },
              "k:{\"type\":\"NamespaceFinalizersRemaining\"}": {
                ".": {},
                "f:lastTransitionTime": {},
                "f:message": {},
                "f:reason": {},
                "f:status": {},
                "f:type": {}
              }
            }
          }
        },
        "subresource": "status"
      }
    ]
  },
  "spec": {},
  "status": {
    "phase": "Terminating",
    "conditions": [
      {
        "type": "NamespaceDeletionDiscoveryFailure",
        "status": "True",
        "lastTransitionTime": "2022-03-21T04:18:56Z",
        "reason": "DiscoveryFailed",
        "message": "Discovery failed for some groups, 1 failing: unable to retrieve                                                                                         the complete list of server APIs: metrics.k8s.io/v1beta1: the server is currently un                                                                                        able to handle the request"
      },
      {
        "type": "NamespaceDeletionGroupVersionParsingFailure",
        "status": "False",
        "lastTransitionTime": "2022-03-21T04:18:57Z",
        "reason": "ParsedGroupVersions",
        "message": "All legacy kube types successfully parsed"
      },
      {
        "type": "NamespaceDeletionContentFailure",
        "status": "False",
        "lastTransitionTime": "2022-03-21T04:18:57Z",
        "reason": "ContentDeleted",
        "message": "All content successfully deleted, may be waiting on finalization                                                                                        "
      },
      {
        "type": "NamespaceContentRemaining",
        "status": "False",
        "lastTransitionTime": "2022-03-21T04:18:57Z",
        "reason": "ContentRemoved",
        "message": "All content successfully removed"
      },
      {
        "type": "NamespaceFinalizersRemaining",
        "status": "False",
        "lastTransitionTime": "2022-03-21T04:18:57Z",
        "reason": "ContentHasNoFinalizers",
        "message": "All content-preserving finalizers finished"
      }
    ]
  }
}
```
可以看到最後的 Terminating 的區塊有解釋原因。

## 解決方法
按照原因解決即可成功刪除 namespace，本文遇到的問題是因為安裝 metric server 時失敗，可以看到 apiservice 其中有出現 false 狀態。
```
kubectl get apiservice
NAME                                   SERVICE                      AVAILABLE                  AGE
v1.                                    Local                        True                       9d
v1.admissionregistration.k8s.io        Local                        True                       9d
v1.apiextensions.k8s.io                Local                        True                       9d
v1.apps                                Local                        True                       9d
v1.authentication.k8s.io               Local                        True                       9d
v1.authorization.k8s.io                Local                        True                       9d
v1.autoscaling                         Local                        True                       9d
v1.batch                               Local                        True                       9d
v1.certificates.k8s.io                 Local                        True                       9d
v1.coordination.k8s.io                 Local                        True                       9d
v1.crd.projectcalico.org               Local                        True                       9d
v1.discovery.k8s.io                    Local                        True                       9d
v1.events.k8s.io                       Local                        True                       9d
v1.monitoring.coreos.com               Local                        True                       3d23h
v1.networking.k8s.io                   Local                        True                       9d
v1.node.k8s.io                         Local                        True                       9d
v1.policy                              Local                        True                       9d
v1.rbac.authorization.k8s.io           Local                        True                       9d
v1.scheduling.k8s.io                   Local                        True                       9d
v1.storage.k8s.io                      Local                        True                       9d
v1alpha1.kafka.strimzi.io              Local                        True                       3h3m
v1alpha1.monitoring.coreos.com         Local                        True                       3d23h
v1beta1.batch                          Local                        True                       9d
v1beta1.discovery.k8s.io               Local                        True                       9d
v1beta1.events.k8s.io                  Local                        True                       9d
v1beta1.flowcontrol.apiserver.k8s.io   Local                        True                       9d
v1beta1.kafka.strimzi.io               Local                        True                       3h3m
v1beta1.metrics.k8s.io                 kube-system/metrics-server   False (MissingEndpoints)   3d1h
v1beta1.node.k8s.io                    Local                        True                       9d
v1beta1.policy                         Local                        True                       9d
v1beta1.storage.k8s.io                 Local                        True                       9d
v1beta2.core.strimzi.io                Local                        True                       3h3m
v1beta2.flowcontrol.apiserver.k8s.io   Local                        True                       9d
v1beta2.kafka.strimzi.io               Local                        True                       3h3m
v2.autoscaling                         Local                        True                       9d
v2beta1.autoscaling                    Local                        True                       9d
v2beta2.autoscaling                    Local                        True                       9d
```
先暫時移除臨時裝的 metric server
```
helm uninstall metrics-server --namespace kube-system
```
就可以成功 delete 了
```
[root@node ~]# kubectl create ns test
namespace/test created
[root@node ~]#
[root@node ~]# kubectl delete ns test
namespace "test" deleted
[root@node ~#
```

</br>
{{< notice warning >}}
另外關於 metric-server 的 debug，紀錄在另外一篇 <a href="https://ulagraphy.netlify.app/post/k8s-helm-install-metrics-server/kubernets-install-metrics-server-by-helm/">文章</a>。
{{< /notice >}}