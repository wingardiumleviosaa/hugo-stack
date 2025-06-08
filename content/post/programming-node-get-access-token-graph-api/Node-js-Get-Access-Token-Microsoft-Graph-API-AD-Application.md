---
title: "[Node.js] Get Access Token 並使用 Microsoft Graph API 操作 AD Application"
tags:
  - Node.js
categories:
  - Programming
date: 2022-10-23 18:55:00
slug: node-get-azure-graph-access-token
---

## 情景

使用 NodeJS `@azure/msal-node` module 取得 Access Token，並進一步使用 Microsoft Graph API 對操作 Azure AD Application 做 Role Assignment。

<!--more-->

## 作法

```js
import * as https from "https";
import fetch from "node-fetch";
import * as msal from "@azure/msal-node";

const httpsAgent = new https.Agent({
  rejectUnauthorized: false,
  keepAlive: true,
});

const msalConfig = {
  auth: {
    clientId: "<YOUR CLIENT ID>",
    clientSecret: "<YOUR CLIENT SECRET>",
    authority: "https://login.microsoftonline.com/<YOUR TENANT ID>/",
  },
};

async function getToken(): Promise<any> {
  const cca = new msal.ConfidentialClientApplication(msalConfig);
  try {
    return await cca.acquireTokenByClientCredential({
      scopes: ["https://graph.microsoft.com/.default"],
    });
  } catch (e) {
    console.log(`Get Azure Access Token Error: ${e}`);
  }
}

async function addAzureOpsgenieAppRole(
  userPrincipalName: string
): Promise<any> {
  const authResponse = await getToken();
  const response = await fetch(
    `https://graph.microsoft.com/v1.0/users/${userPrincipalName}`,
    {
      method: "get",
      headers: { Authorization: `Bearer ${authResponse.accessToken}` },
      agent: httpsAgent,
    }
  )
    .then((response) => {
      if (response.ok) {
        return response.json();
      } else {
        return Promise.reject(response);
      }
    })
    .then((data) => {
      const createBody = {
        principalId: data.id,
        resourceId: "<YOUR RESOURCE ID>",
        appRoleId: "<YOUR APPLICATION ROLE ID>",
      };
      return fetch(
        `https://graph.microsoft.com/v1.0/users/${data.id}/appRoleAssignments`,
        {
          method: "post",
          body: JSON.stringify(createBody),
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${authResponse.accessToken}`,
          },
          agent: httpsAgent,
        }
      ).then((response) => {
        if (response.ok) {
          console.log(
            "Successfully add user as assgined role to Azure AD Application"
          );
          return response.json();
        } else {
          return Promise.reject(response);
        }
      });
    })
    .catch((e) => {
      console.warn("User Assignment already exists or error happens", e);
    });
}

addAzureOpsgenieAppRole("ulahsieh@abc.com");
```
