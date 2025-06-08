---
title: >-
  [Terraform] azurerm_key_vault_secret Error: Provider produced inconsistent
  result after apply
tags:
  - Terraform
categories:
  - SRE
date: 2022-09-21 20:31:00
slug: terraform-azurerm-key-vault-bug
---

## 問題描述

使用 Terraform 的 azurerm provider 創建 key vault secret 時，會出現以下錯誤：

<!--more-->

```
│ Error: Provider produced inconsistent result after apply
│
│ When applying changes to azurerm_key_vault_secret.sbcrptnid_srt, provider
│ "provider[\"registry.terraform.io/hashicorp/azurerm\"]" produced an unexpected
│ new value: Root resource was present, but now absent.
│
│ This is a bug in the provider, which should be reported in the provider's own
│ issue tracker.
```

雖然報錯，但 secret 資源有成功被創建。該結果也可以在第二次 apply 後發現到，會有不同錯誤訊息指出該 resource 已經存在的錯誤：

```
╷
│ Error: A resource with the ID "https://kv-srt-test.vault.azure.net/secrets/test/458f9046eeef4ae08a114xxxxxxxxxx" already exists - to be managed via Terraform this resource needs to be imported into the State. Please see the resource documentation for "azurerm_key_vault_secret" for more information.
│
│   with azurerm_key_vault_secret.sbcrptnid_srt,
│   on main.tf line 5, in resource "azurerm_key_vault_secret" "sbcrptnid_srt":
│    5: resource "azurerm_key_vault_secret" "sbcrptnid_srt" {
│
```

為了完善自動化流程，打算透過 `terraform import` 的手法將建成功但沒記錄在 terraform state file 的 secret 匯入，但匯入時會報錯：

```
Error: Cannot import non-existent remote object
│
│ While attempting to import an existing object to
│ "azurerm_key_vault_secret.sbcrptnid_srt", the provider detected that no object
│ exists with the given id. Only pre-existing objects can be imported; check that
│ the id is correct and that it is associated with the provider's configured
│ region or endpoint, or use "terraform apply" to create a new remote object for
│ this resource.
```

</br>

![](https://imgur.com/qCRnVwT.png)

## 原因

該錯誤參考 azurerm 的 [gitlab issue #11059](https://github.com/hashicorp/terraform-provider-azurerm/issues/11059#)，可以發現到如錯誤訊息所描述的是 provider 的 bug。

## 解決方法

目前 provider 尚未 close 掉該 bug，但可以變相從 terraform import 去更新 state file，解決 apply 都會報錯的問題。參考該 issue 的討論，原本試著用 community 提供的為 key vault 加上 tag 的方式，但失敗，後來在同樣的 issue 下看到解決方式：
用 `az account set -s ...` 先指定 key vault 的 subscription，接著再下 `terraform import`，就可以成功匯入 secret 至 state file。

![](https://imgur.com/nWDz42k.png)

最後照常下 terraform apply 就都不會報錯了。

![](https://imgur.com/smg1VI8.png)
