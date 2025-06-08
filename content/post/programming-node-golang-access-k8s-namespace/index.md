---
title: "[Golang & Node.js] Access Kubernetes Out of the Cluster"
tags:
  - Golang
  - Node.js
  - Kubernetes
categories:
  - Programming
  - Kubernetes
date: 2023-06-05 17:19:00
slug: golang-and-nodejs-client-access-k8s
---

<!--more-->

## Prepare .kube/config

使用 Rancher 產生好帶有 user token 的 kubeconfig 檔案存取，將 config 檔下載後放置與 code 同層的目錄下。

![](https://imgur.com/f9EoEL4.png)

## Go Clinet

查詢指定的 namespace 是否存在

```go
package main

import (
	"context"
	"fmt"

	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// uses the current context in kubeconfig
	// path-to-kubeconfig -- for example, /root/.kube/config
	config, _ := clientcmd.BuildConfigFromFlags("", "./config")
	// creates the clientset
	clientset, _ := kubernetes.NewForConfig(config)
	ns, err := clientset.CoreV1().Namespaces().Get(context.TODO(), "mynamespace", metav1.GetOptions{})
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("namespace exist: ", ns)
	}
}
```

## JavaScript Client

查詢指定的 namespace 是否存在

版本一：

```js
try {
  await k8sApi.readNamespace("mynamespace").then(
    (res) => {
      if (res.response.statusCode === 200 || res.response.statusCode === 201) {
        status = "synced";
        logger.info(`mynamespace namespace 成功 sync 至 Kubernetes Cluster`);
      }
    },
    (err) => {
      status = "sync failed";
      logger.error(
        `Kubernetes Cluster 無該 mynamespace namespace: ${err.body}`
      );
    }
  );
  return status;
} catch (e) {
  logger.error(`k8s 連線錯誤: ${e}`);
  return status;
}
```

版本二：

```js
const k8s = require("@kubernetes/client-node");

process.env["NODE_TLS_REJECT_UNAUTHORIZED"] = "0";
const kc = new k8s.KubeConfig();
kc.loadFromFile("./config");

const k8sApi = kc.makeApiClient(k8s.CoreV1Api);

let nslist = [];

async function getNamespace(checkNs: string) {
  await k8sApi.listNamespace().then((res) => {
    res.body.items.forEach((ns) => nslist.push(ns.metadata.name));
  });
  // console.log(nslist)

  for (let i of nslist) {
    if (i === checkNs) {
      console.log("namespace created! sync success!", i);
    }
  }
}
```

## Refernece

- https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#programmatic-access-to-the-api
- [首圖圖片來源](https://nordicapis.com/wp-content/uploads/Best-Practices-for-Hosting-an-API-on-Kubernetes-1024x576.png)
