---
title: "[Terraform] Upgrade to the Latest Version"
tags:
  - Terraform
categories:
  - SRE
date: 2021-09-14 13:02:00
slug: terraform-upgrage-to-v1
---

目前的環境使用的 terraform 版本是 v0.13.5

![](https://imgur.com/QDUkUEk.png)

參考以下官網說明，如果從 0.13 版要往最新版 v1 升，需要先升到 0.14。

<!--more-->

| Current Version  | Recommendation                                                                                                                                                                                                                   |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| v0.10 or earlier | Refer to the upgrade guides for these historical versions until you have upgraded to the latest v0.11 release, then refer to the following item.                                                                                 |
| v0.11            | Use the terraform 0.12checklist command to detect any situations that must be addressed before upgrading to v0.12, resolve them, and then upgrade to the latest v0.12 release and follow the v0.12 Upgrade Guide.                |
| v0.12            | Upgrade to the latest Terraform v0.13 release and then follow the v0.13 upgrade guide to upgrade your configuration and state for explicit provider requirements.                                                                |
| v0.13            | Upgrade to the latest Terraform v0.14 release and attempt a normal Terraform run. If you encounter any new errors, refer to the v0.14 upgrade guide for resolution steps.                                                        |
| v0.14            | Upgrade directly to the latest Terraform v1.0 release and attempt a normal Terraform run. If you encounter any new errors, refer to the v0.15 upgrade guide for resolution steps.                                                |
| v0.15            | Upgrade directly to the latest Terraform v1.0 release and attempt a normal Terraform run. Terraform v1.0 is a continuation of the v0.15 series, and so v1.0.0 and later are directly backward-compatible with Terraform v0.15.5. |

## 開始升級

各版 release [載點](https://releases.hashicorp.com/terraform/)，請依照不同的作業系統安裝。

![](https://imgur.com/Fbo6jfu.png)

```bash
[root@master1 ~]# wget https://releases.hashicorp.com/terraform/0.14.11/terraform_0.14.11_linux_amd64.zip
--2021-09-14 13:00:30--  https://releases.hashicorp.com/terraform/0.14.11/terraform_0.14.11_linux_amd64.zip
Resolving releases.hashicorp.com (releases.hashicorp.com)... 151.101.1.183, 151.101.65.183, 151.101.129.183, ...
Connecting to releases.hashicorp.com (releases.hashicorp.com)|151.101.1.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 33789439 (32M) [application/zip]
Saving to: ‘terraform_0.14.11_linux_amd64.zip’

100%[==============================================================================================>] 33,789,439  8.05MB/s   in 4.0s

2021-09-14 13:00:34 (8.10 MB/s) - ‘terraform_0.14.11_linux_amd64.zip’ saved [33789439/33789439]

[root@master1 ~]#
[root@master1 ~]# unzip terraform_0.14.11_linux_amd64.zip
Archive:  terraform_0.14.11_linux_amd64.zip
  inflating: terraform

[root@master1 ~]#
[root@master1 ~]# mv terraform /usr/local/bin/
mv: overwrite ‘/usr/local/bin/terraform’? y
[root@master1 ~]# terraform version
Terraform v0.14.11

Your version of Terraform is out of date! The latest version
is 1.0.6. You can update by downloading from https://www.terraform.io/downloads.html
[root@master1 ~]#
[root@master1 ~]# wget https://releases.hashicorp.com/terraform/1.0.6/terraform_1.0.6_linux_amd64.zip
[root@master1 ~]# wget https://releases.hashicorp.com/terraform/1.0.6/terraform_1.0.6_linux_amd64.zip
--2021-09-14 13:31:21--  https://releases.hashicorp.com/terraform/1.0.6/terraform_1.0.6_linux_amd64.zip
Resolving releases.hashicorp.com (releases.hashicorp.com)... 151.101.129.183, 151.101.1.183, 151.101.65.183, ...
Connecting to releases.hashicorp.com (releases.hashicorp.com)|151.101.129.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 32677516 (31M) [application/zip]
Saving to: ‘terraform_1.0.6_linux_amd64.zip’

100%[=================================================================================================================>] 32,677,516  11.0MB/s   in 2.8s

2021-09-14 13:31:24 (11.0 MB/s) - ‘terraform_1.0.6_linux_amd64.zip’ saved [32677516/32677516]

[root@master1 ~]#
[root@master1 ~]# unzip terraform_1.0.6_linux_amd64.zip
Archive:  terraform_1.0.6_linux_amd64.zip
  inflating: terraform

[root@master1 ~]#
[root@master1 ~]# mv terraform /usr/local/bin/
mv: overwrite ‘/usr/local/bin/terraform’? y
[root@master1 ~]# terraform version
Terraform v1.0.6
```

## cannot execute binary file

如果在安裝完後，使用 `terrform` cli 時，發現以下錯誤

```
[root@master1 ~]# terraform version
-bash: /usr/local/bin/terraform: cannot execute binary file
```

則是因為下載到錯誤的作業系統的安裝包。

## Reference

- https://www.terraform.io/upgrade-guides/1-0.html
