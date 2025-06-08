---
title: "[Azure] 使用 Vault Secrets API 如何取得超過 25 筆後的資料"
tags:
  - Azure
categories:
  - SRE
date: 2023-02-18 16:32:00
slug: get-azure-vault-secret-for-all
---

## TL; DR

使用 Azure key vault 的 API Get 所有 secrets 時可以利用 `maxresults` 的參數限制回傳的筆數，預設為 25 筆，但經測試，最大的顯示筆數也就是 25 筆了，要如何取得全部的 secrets 呢? 以此篇記錄。

<!--more-->

## 利用 nextLink 取得下一頁資料

假設目前的 vault URI 為 `https://arm-vault-prd.vault.azure.net/` 的，我們先來看最一開始使用 get secrets 返回的結果，因不想一次顯示 25 筆，利用 `maxresults` 參數指令顯示兩筆：

```json
{
  "value": [
    {
      "id": "https://arm-vault-prd.vault.azure.net/secrets/azure-AZRAutoTesting-prd-azure-client-id",
      "attributes": {
        "enabled": true,
        "created": 1651506090,
        "updated": 1651506090,
        "recoveryLevel": "Recoverable+Purgeable",
        "recoverableDays": 90
      },
      "tags": {}
    },
    {
      "id": "https://arm-vault-prd.vault.azure.net/secrets/azure-AZRAutoTesting-prd-azure-client-secret",
      "attributes": {
        "enabled": true,
        "created": 1651506109,
        "updated": 1651506109,
        "recoveryLevel": "Recoverable+Purgeable",
        "recoverableDays": 90
      },
      "tags": {}
    }
  ],
  "nextLink": "https://arm-vault-prd.vault.azure.net:443/secrets?api-version=7.4&$skiptoken=eyJOZXh0TWFya2VyIjoiMiExMzIhTURBd01EVXpJWE5sWTNKbGRDOUJXbFZTUlMxQldsSkJWVlJQVkVWVFZFbE9SeTFRVWtRdFFWcFZVa1V0VTFWQ1UwTlNTVkJVU1U5T0xVbEVJVEF3TURBeU9DRTVPVGs1TFRFeUxUTXhWREl6T2pVNU9qVTVMams1T1RrNU9UbGFJUS0tIiwiVGFyZ2V0TG9jYXRpb24iOjB9&maxresults=2"
}
```

可以發現回傳的資訊最後有一個 nextLink 的參數，這個就是下兩筆 secrets 取得的位址！請注意使用此 API 一樣要在 header 輸入 Authencation 的資訊。
用 Postman 試打的結果如下：  
第一次:

![](https://imgur.com/0fQwegU.png)

第二次使用第一次回傳的 NextLink:

![](https://imgur.com/KkOtXd5.png)
