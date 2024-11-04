---
title: 'Ingress does not contain a valid IngressClass'
categories: ["Kubernetes"]
tags: ["Kubernetes", "Ingress"]
date: 2022-02-11 06:20:00
slug: kubernets-ingress-invalid-ingressclass
---
## 問題描述
nginx ingress 從原本 deprecated 的 stable/nginx-ingress helm chart 改為 ingress-nginx/ingress-nginx chart 後，發現 ingress resource 的 nginx 網頁 404 not found，查看 ingress nginx controller log 發現有 `ingress does not contain a valid IngressClass` 的錯誤。
<!--more-->

![](https://imgur.com/lPUFtBq.png)

完整 log 如下：
```
[root@testm terraform]# kubectl logs -f -n ingress-nginx ingress-nginx-controller-5c5bf8c854-7pcf7
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:       v1.1.1
  Build:         a17181e43ec85534a6fea968d95d019c5a4bc8cf
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.9

-------------------------------------------------------------------------------

W0209 05:43:07.359176       7 client_config.go:615] Neither --kubeconfig nor --master was specified.  Using thesterConfig.  This might not work.
I0209 05:43:07.359883       7 main.go:223] "Creating API client" host="https://10.96.0.1:443"
I0209 05:43:07.379993       7 main.go:267] "Running in Kubernetes cluster" major="1" minor="20" git="v1.20.15" "clean" commit="8f1e5bf0b9729a899b8df86249b56e2c74aebc55" platform="linux/amd64"
I0209 05:43:07.644539       7 main.go:104] "SSL fake certificate created" file="/etc/ingress-controller/ssl/defake-certificate.pem"
I0209 05:43:07.675915       7 ssl.go:531] "loading tls certificate" path="/usr/local/certificates/cert" key="/ual/certificates/key"
I0209 05:43:07.727278       7 nginx.go:255] "Starting NGINX Ingress controller"
I0209 05:43:07.828276       7 event.go:282] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginxe:"ingress-nginx-controller", UID:"c6943575-18fe-471a-aa44-5f251af7a277", APIVersion:"v1", ResourceVersion:"104FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/ingress-nginx-controller
I0209 05:43:08.929877       7 nginx.go:297] "Starting NGINX process"
I0209 05:43:08.930002       7 leaderelection.go:248] attempting to acquire leader lease ingress-nginx/ingress-cler-leader...
I0209 05:43:08.930504       7 nginx.go:317] "Starting validation webhook" address=":8443" certPath="/usr/local/icates/cert" keyPath="/usr/local/certificates/key"
I0209 05:43:08.930926       7 controller.go:155] "Configuration changes detected, backend reload required"
I0209 05:43:08.945063       7 leaderelection.go:258] successfully acquired lease ingress-nginx/ingress-controllder
I0209 05:43:08.945130       7 status.go:84] "New leader elected" identity="ingress-nginx-controller-5c5bf8c854-
I0209 05:43:09.015595       7 controller.go:172] "Backend successfully reloaded"
I0209 05:43:09.015708       7 controller.go:183] "Initial sync, sleeping for 1 second"
I0209 05:43:09.015766       7 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Namress-nginx-controller-5c5bf8c854-7pcf7", UID:"f84b09e4-7d7d-40bf-ae66-f2eb72ab7a59", APIVersion:"v1", ResourceV:"104846", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configurat
W0209 05:43:26.721706       7 controller.go:988] Error obtaining Endpoints for Service "monitoring/prometheus-or-prometheus": no object matching key "monitoring/prometheus-operator-prometheus" in local store
W0209 05:43:26.721831       7 controller.go:1083] Service "monitoring/prometheus-operator-grafana" does not havactive Endpoint.
W0209 05:43:26.722981       7 controller.go:988] Error obtaining Endpoints for Service "monitoring/prometheus-or-alertmanager": no object matching key "monitoring/prometheus-operator-alertmanager" in local store
I0209 05:43:26.781310       7 admission.go:149] processed ingress via admission controller {testedIngressLengthtedIngressTime:0.06s renderingIngressLength:1 renderingIngressTime:0.001s admissionTime:18.0kBs testedConfiguraze:0.061}
I0209 05:43:26.781370       7 main.go:101] "successfully validated configuration, accepting" ingress="monitorinetheus-ingress"
I0209 05:43:26.784116       7 admission.go:149] processed ingress via admission controller {testedIngressLengthtedIngressTime:0.063s renderingIngressLength:1 renderingIngressTime:0s admissionTime:18.0kBs testedConfiguratio0.063}
I0209 05:43:26.784155       7 main.go:101] "successfully validated configuration, accepting" ingress="monitorinana-ingress"
I0209 05:43:26.785125       7 admission.go:149] processed ingress via admission controller {testedIngressLengthtedIngressTime:0.063s renderingIngressLength:1 renderingIngressTime:0.002s admissionTime:18.1kBs testedConfigurize:0.065}
I0209 05:43:26.785171       7 main.go:101] "successfully validated configuration, accepting" ingress="monitorintmanager-ingress"
I0209 05:43:26.785238       7 admission.go:149] processed ingress via admission controller {testedIngressLengthtedIngressTime:0.064s renderingIngressLength:1 renderingIngressTime:0s admissionTime:18.0kBs testedConfiguratio0.064}
I0209 05:43:26.785259       7 main.go:101] "successfully validated configuration, accepting" ingress="istio-sysali-ingress"
I0209 05:43:26.931088       7 store.go:420] "Ignoring ingress because of error while validating ingress class" s="monitoring/grafana-ingress" error="ingress does not contain a valid IngressClass"
I0209 05:43:26.931132       7 store.go:420] "Ignoring ingress because of error while validating ingress class" s="monitoring/prometheus-ingress" error="ingress does not contain a valid IngressClass"
I0209 05:43:26.931149       7 store.go:420] "Ignoring ingress because of error while validating ingress class" s="monitoring/alertmanager-ingress" error="ingress does not contain a valid IngressClass"
I0209 05:43:26.945691       7 store.go:420] "Ignoring ingress because of error while validating ingress class" s="istio-system/kiali-ingress" error="ingress does not contain a valid IngressClass"
```


## 原 Ingress 配置
以 `grafana-ing.tf` 為範例 (使用 terraform 自動部署工具安裝)
```r
resource "kubernetes_ingress" "grafana_ingress" {
  count = var.package.prometheus == true ? 1 : 0

  metadata {
    name =      "grafana-ingress"
    namespace = "monitoring"
  }

  spec {
    rule {
      host = "${var.prometheus-options.host_grafana}.${var.domain}"
      http {
        path {
          backend {
            service_name = "prometheus-operator-grafana"
            service_port = 80
          }
          path = "/"
        }
      }
    }
  }

  depends_on = [
    helm_release.ingress-nginx,
  ]
}
```

## 發生原因
原因是在 ingress v1.0.0 版之後，ingress 需要加上 ingress class，請參考 github 的 [#7341](https://github.com/kubernetes/ingress-nginx/pull/7341) pull request，如果沒有，controller 會丟 `Ignoring ingress because of error while validating ingress class" ingress="k8sNamespace/ingressResourceName" error="ingress does not contain a valid IngressClass"` 的錯誤。

{{< notice info >}}
An Ingress Class is basically a category which specify who needs to serve and manage the Ingress, this is necessary since in a cluster you can have more than one Ingress controller, each one with its rules and configurations.
{{< /notice >}}

## 解決方法
在 ingress resource 中的 metadata 欄位加上 `annotations: kubernetes.io/ingress.class: "nginx"` 即可。

```r
resource "kubernetes_ingress" "grafana_ingress" {
  count = var.package.prometheus == true ? 1 : 0

  metadata {
    name =      "grafana-ingress"
    namespace = "monitoring"
    annotations = {
      "kubernetes.io/ingress.class" = "nginx"
    }
  }

  spec {
    rule {
      host = "${var.prometheus-options.host_grafana}.${var.domain}"
      http {
        path {
          backend {
            service_name = "prometheus-operator-grafana"
            service_port = 80
          }
          path = "/"
        }
      }
    }
  }

  depends_on = [
    helm_release.ingress-nginx,
  ]
}
```

## 注意
上述方法僅適用於 kubernetes 1.22 版以前。k8s 1.22 版以後請使用 `ingressClassName` 於 spec 區塊下。請參考[官網](https://kubernetes.github.io/ingress-nginx/user-guide/multiple-ingress/)說明。

## Reference
- https://forum.linuxfoundation.org/discussion/859965/exercise-7-nginx-update-requires-change-to-yaml
- https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/ingress
- https://stackoverflow.com/questions/65979766/ingress-with-nginx-controller-not-working-address-missing