---
title: 'Change MetalLB IP Range'
categories: ["Kubernetes"]
tags: ["Kubernetes", "MetalLB"]
date: 2021-09-16 10:30:00
slug: kubernets-metallb
---


```bash
# note the old IPs allocated to the services
kubectl get svc

# edit config
kubectl edit cm -n metallb-system config

# delete the metallb pods
kubectl -n metallb-system delete pod --all

# watch the pods come back up
kubectl -n metallb-system get pods -w

# inspect new IPs of services
kubectl get svc
```

<!--more-->

or

```bash
cat << EOF > new_config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.0.0.10-10.0.0.19
EOF

# delete the old configmap
kubectl -n metallb-system delete cm config

# apply the new configmap
kubectl apply -f new_config.yaml
```

## why we need to delete all metallb pod

MetalLB rejects new configurations if the new configuration would break existing services. Your new configuration does not allow existing services to continue existing, so MetalLB ignored it. There are log entries in the controller and speaker pods about this.

To force MetalLB to accept an unsafe configuration, delete all the controller and speaker pods. When they restart, they'll accept the new configuration and change all your services. 

`kubectl delete po -n metallb-system --all`

## Reference
- [https://github.com/metallb/metallb/issues/308](https://github.com/metallb/metallb/issues/308)
- [https://github.com/metallb/metallb/issues/348](https://github.com/metallb/metallb/issues/348)