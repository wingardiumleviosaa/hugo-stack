---
title: '[Kubernetes] 安裝高可用 PostgreSQL'
tags:
  - Kubernetes
  - PostgreSQL
categories:
  - Kubernetes
date: 2024-05-25T10:57:04+08:00
slug: k8s-install-ha-postgresql
---

### 建立 PostgreSQL  HA 叢集

直接透過 bitnami 的 postgresql-ha helm chart 安裝

<!--more-->

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

```
可以透過指定 values.yaml 安裝
```
helm install pgsql -f values.yaml bitnami/postgresql-ha -n platform
```
或是直接在安裝時指定要替換的參數，因為要修改的參數只有 storage class，所以直接用此命令安裝
```
helm install pgsql bitnami/postgresql-ha -n platform --set global.storageClass=rook-ceph-block
```

安裝完成後會跳提示，可以取得 DB 的登入密碼，以及說明如何連線。

```bash
To get the password for "postgres" run:
    export POSTGRES_PASSWORD=$(kubectl get secret --namespace platform pgsql-postgresql-ha-postgresql -o jsonpath="{.data.password}" | base64 -d)

To get the password for "repmgr" run:
    export REPMGR_PASSWORD=$(kubectl get secret --namespace platform pgsql-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 -d)

To connect to your database run the following command:
    kubectl run pgsql-postgresql-ha-client --rm --tty -i --restart='Never' --namespace platform --image docker.io/bitnami/postgresql-repmgr:16.0.0-debian-11-r15 --env="PGPASSWORD=$POSTGRES_PASSWORD"  \
        --command -- psql -h pgsql-postgresql-ha-pgpool -p 5432 -U postgres -d postgres

To connect to your database from outside the cluster execute the following commands:
    kubectl port-forward --namespace platform svc/pgsql-postgresql-ha-pgpool 5432:5432 &
    psql -h 127.0.0.1 -p 5432 -U postgres -d postgres
```

### update values.yaml
如果安裝後有需要修改設定，可以再改完 values.yaml 後用 upgrade 指令進行更新。
```yaml
helm upgrade -f values.yaml pgsql bitnami/postgresql-ha -n platform 
```

### 建立 PGADMIN4 UI

postgresql-ha helm chart 不包含 PGADIMN UI，另外使用 yaml 部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pgadmin
  namespace: platform
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pgadmin
  template:
    metadata:
      labels:
        app: pgadmin
    spec:
      containers:
      - env:
        - name: PGADMIN_DEFAULT_EMAIL
          value: asus.sdsp@gmail.com
        - name: PGADMIN_DEFAULT_PASSWORD
          value: "!QAZxsw2"
        - name: PGADMIN_LISTEN_PORT
          value: "8001"
        - name: PGADMIN_CONFIG_WTF_CSRF_CHECK_DEFAULT
          value: "False"
        - name: PGADMIN_CONFIG_WTF_CSRF_ENABLED
          value: "False"
        - name: PGADMIN_CONFIG_ENHANCED_COOKIE_PROTECTION
          value: "False"
        image: dpage/pgadmin4:7.8
        imagePullPolicy: IfNotPresent
        name: pgadmin
        ports:
        - name: tcp8001
          containerPort: 8001
---
kind: Service
apiVersion: v1
metadata:
  namespace: platform
  name: pgadmin
spec:
  type: ClusterIP
  ports:
    - name: http
      protocol: TCP
      port: 8001
      targetPort: tcp8001
  selector:
    app: pgadmin
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pgadmin
  namespace: platform
  annotations:
    kubernetes.io/ingress.class: nginx
    #cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
spec:
  tls:
  - hosts:
    - pgadmin.sdsp-stg.com
    secretName: domain-cert-sdsp-stg.com-prod
  rules:
    - host: pgadmin.sdsp-stg.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: pgadmin
                port:
                  number: 8001
```

### Error: The CSRF tokens do not match

如果 PGADMIN 啟用時遇到以下錯誤

```yaml
2023-11-15 03:03:39,842: ERROR  pgadmin:        400 Bad Request: The CSRF tokens do not match.
Traceback (most recent call last):
  File "/venv/lib/python3.11/site-packages/flask_wtf/csrf.py", line 261, in protect
    validate_csrf(self._get_csrf_token())
  File "/venv/lib/python3.11/site-packages/flask_wtf/csrf.py", line 115, in validate_csrf
    raise ValidationError("The CSRF tokens do not match.")
wtforms.validators.ValidationError: The CSRF tokens do not match.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/venv/lib/python3.11/site-packages/flask/app.py", line 1821, in full_dispatch_request
    rv = self.preprocess_request()
         ^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/venv/lib/python3.11/site-packages/flask/app.py", line 2313, in preprocess_request
    rv = self.ensure_sync(before_func)()
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/venv/lib/python3.11/site-packages/flask_wtf/csrf.py", line 229, in csrf_protect
    self.protect()
  File "/venv/lib/python3.11/site-packages/flask_wtf/csrf.py", line 264, in protect
    self._error_response(e.args[0])
  File "/venv/lib/python3.11/site-packages/flask_wtf/csrf.py", line 307, in _error_response
    raise CSRFError(reason)
```

可以加上三個環境變數解決：

- PGADMIN_CONFIG_WTF_CSRF_CHECK_DEFAULT="False"
- PGADMIN_CONFIG_WTF_CSRF_ENABLED="False"
- PGADMIN_CONFIG_ENHANCED_COOKIE_PROTECTION="False"