---
title: "[Azure] 透過 Managment API 建立訂閱帳戶"
tags:
  - Azure
categories:
  - SRE
date: 2023-02-14 16:32:00
image: "images.png"
slug: create-subscription-by-azure-mgmt-api-and-spn
---

## TL;DR

本篇文章記錄透過 Azure Management API 使用 Azure Service Principal 權限授權，建立 Billing Account 類型為 Enterprise Agreement 的 Subscription 訂閱帳戶。

<!--more-->

## 先決條件: 賦予權限

在 Azure Portal 上沒有針對 Billing Account 的 IAM 可以設定，需要透過 API 賦予 `subscription creator role` 給對應的服務主體(service principal)。這個步驟可以透過 Microsoft Learn 提供的 API focus mode 功能實作，請前往[enrollment-account-role-assignments 的 API 文檔](https://learn.microsoft.com/en-us/rest/api/billing/2019-10-01-preview/enrollment-account-role-assignments/put?tabs=HTTP)。

![](https://i.imgur.com/MRgffVq.png)

### 權限列表

附上在 EA 收費下，SPN 有四種權限可被賦予，一個 SPN 只能被賦予一種權限。

|        Role         |          Role definition ID          |                                                                                                                                                                          Actions allowed                                                                                                                                                                           |
| :-----------------: | :----------------------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|  EnrollmentReader   | 24f8edb6-1668-4659-b5e2-40bb5f3a7d7e |                                Enrollment readers can view data at the enrollment, department, and account scopes. The data contains charges for all of the subscriptions under the scopes, including across tenants. Can view the Azure Prepayment (previously called monetary commitment) balance associated with the enrollment.                                |
|    EA purchaser     | da6647fb-7651-49ee-be91-c43c4877f0c4 | Purchase reservation orders and view reservation transactions. It has all the permissions of EnrollmentReader, which will in turn have all the permissions of DepartmentReader. It can view usage and charges across all accounts and subscriptions. Can view the Azure Prepayment (previously called monetary commitment) balance associated with the enrollment. |
|  DepartmentReader   | db609904-a47f-4794-9be8-9bd86fbffd8a |                                                                                                                  Download the usage details for the department they administer. Can view the usage and charges associated with their department.                                                                                                                   |
| SubscriptionCreator | a0bcee42-bf30-4d1b-926a-48d21664ef71 |                                                                                                                                                      Create new subscriptions in the given scope of Account.                                                                                                                                                       |

## 透過 REST API 建立訂閱帳戶

透過 [Subscription 的 Alias API](https://learn.microsoft.com/en-us/rest/api/subscription/2020-09-01/alias/create?tabs=HTTP)

```json
PUT https://management.azure.com/providers/Microsoft.Subscription/aliases/aliasForNewSub?api-version=2020-09-01

{
  "properties": {
    "displayName": "Contoso MCA subscription",
    "billingScope": "/providers/Microsoft.Billing/billingAccounts/e879cf0f-2b4d-5431-109a-f72fc9868693:024cabf4-7321-4cf9-be59-df0c77ca51de_2019-05-31/billingProfiles/PE2Q-NOIT-BG7-TGB/invoiceSections/MTT4-OBS7-PJA-TGB",
    "workload": "Production"
  }
}
```

</br>

{{< notice info >}}
Azure 的 aliases 資源類型是擴展資源 (extension resource)，可以將其應用於另一個資源。
{{< /notice >}}

## Implement by NodeJS

```js
public async createSubscription(subscriptionName: string, envWorkload: string, envBillingScope: string): Promise<string> {
    let subscriptionId = '';
    try{
      const authResponse = await this.getMgmtToken();
      const createBody = {
        properties: {
          billingScope: envBillingScope,
          displayName: subscriptionName,
          workload: envWorkload,
        },
      };

      const response = await fetch(
        `${this.conf.subscriptionsApi.aliasUrl}/${subscriptionName}${this.conf.subscriptionsApi.aliasUrlVersion}`,
        {
          method: 'put',
          body: JSON.stringify(createBody),
          headers: {'Content-Type': 'application/json', Authorization: `Bearer ${authResponse.accessToken}`},
        },
      )

      if (!response.ok) {
        throw new Error(`建立 subscription 發生錯誤, status: ${response.status}, errorMsg: ${JSON.stringify(await response.json())}`);
      }

      const body : any = await response.json();

      subscriptionId = Object(Object(body).properties).subscriptionId;

    }catch(e){
      logger.error(e)
    }
    return subscriptionId;
  }
```

## Reference

- https://learn.microsoft.com/en-us/azure/templates/microsoft.subscription/aliases?pivots=deployment-language-bicep
- https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/assign-roles-azure-service-principals#assign-the-subscription-creator-role-to-the-spn
- [首圖圖片來源](https://www.bluebridge.lt/it-ziniu-centras/wp-content/uploads/2016/07/windows-azure-cloud.png)
