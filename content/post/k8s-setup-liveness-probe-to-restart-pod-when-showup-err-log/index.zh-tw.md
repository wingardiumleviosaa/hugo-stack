---
title: '設定 liveness probe 監聽應用以重啟 pod'
tags:
  - Kubernetes
categories:
  - Kubernetes
date: 2022-03-31 22:52:15
slug: setup-liveness-probe-to-restart-when-pod-has-error-log
---
目前手上有一個監聽 Oracle CDC 的程式跑在以 Debian 為基底的 kubernetes pod 中，會定期因為 Oracle 的錯誤訊息 ORA-12518: TNS 監聽程式無法分發客戶機連線的問題而斷線。此時雖然程式有 error log，但 Pod 的狀態仍然為 Running，只要重啟 Pod 即可重新正常運作。 

<!--more-->

## ORA-12518

![](https://imgur.com/LnkfICA.png)

首先順便解釋此錯誤的原因 The process of handing off a client connection to another process failed.
參考網路上其他分享：
- [stackoverflow](https://stackoverflow.com/questions/13624464/ora-12518-tnslistener-could-not-hand-off-client-connection)
- [ittutorial](https://ittutorial.org/ora-12518-tns-listener-could-not-hand-off-client-connection/)
- [cnblogs](https://www.cnblogs.com/javadu/archive/2012/02/20/2359556.html)

從根本可以解決的方式如下：
1. Edit /etc/systemd/system.conf file and Set DefaultTasksMax to ‘infinity’.
2. dedicated server: 修改 oracle processes & sessions parameters
3. shared server: 修改 oracle dispatcher parameters

## 然而
因為 IT server 並不在我控管的範圍，所以只能自己手動重啟 Pod。原本是想說寫 cronJob 定期重啟 pod，但在找資料的過程中，發現在 [stackoverflow crobjob 問題](https://stackoverflow.com/a/61328816/13318115) 的解法中有人提出了直接使用 livenessprobe 解決。

## 從設定 livenessProbe 解決
Kubelet 使用 liveness probe（存活探針）來確定何時重啟容器。當應用程序處於運行狀態但無法做進一步操作，liveness 探針將捕獲到 deadlock，重啟處於該狀態下的容器，使應用程序在存在 bug 的情況下依然能夠繼續運行下去。

- exec.Command：要在容器內執行的檢測命令，如果命令執行成功，將返回 0，kubelet 就會認為該容器是活著的並且很健康。如果返回非 0 值，kubelet 就會殺掉這個容器並重啟它。
- periodSeconds：liveness probe 多久檢查一次
- initialDelaySeconds：首次啟動 pod 後，要延遲多久後執行 liveness probe

## probe command 要寫啥?
接下來又另一個問題來了，我的 probe 中的檢測命令要寫啥? 因為在手動重啟時，只能從 kubectl logs 為依據，查看有無錯誤訊息。然而現在 command 要執行在容器中，但容器裡面沒辦法直接使用 kubectl 取得應用的 stdout 的訊息。又去堆疊溢位(XD 找到了兩種解決方法。

### 在容器裡 curl Kubernete API server
設定 pod 連 kubernetes api server 請參考另外一篇[文章記錄](https://ulahsieh.netlify.app/p/curl-kubernetes-api-server-within-pod/)。
command 應該就會長成以下，如果 curl 回到的 output 會 grep 到 error 訊息，則重啟。
```yaml
livenessProbe:
  exec:
    command:
    - bash
    - -c
    - "curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt --header "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" -X GET https://kubernetes.default.svc/api/v1/namespaces/converg-it/pods/converg-it-adapter-oracle-8575f54dc6-7lnv6/log?sinceSeconds=100 | grep 'error'"
  initialDelaySeconds: 120
  periodSeconds: 60
```

### 在容器裡查看監聽的 port
在目前跑的容器中，使用 ss 查看目前系統的 socket 狀態，可以發現到其實在正常連結的情況下能偵測到連線(establish state) oracle server 的監聽。

![](https://imgur.com/YAgQZeI.png)

那麼當發生連線異常時(ORA-12518)，就可以當作是重啟的條件。

![](https://imgur.com/s8QhlJT.png)
```yaml
livenessProbe:
  exec:
    command:
    - bash
    - -c
    - "ss -an | grep -q 'EST.*:1521 *$'"
  initialDelaySeconds: 120
  periodSeconds: 60
```

## Reference
- https://jimmysong.io/kubernetes-handbook/guide/configure-pod-service-account.html
- https://stackoverflow.com/questions/49000280/monitor-and-take-action-based-on-pod-log-event
- https://stackoverflow.com/questions/57711963/kubernetes-liveness-probe-can-a-pod-monitor-its-own-stdout