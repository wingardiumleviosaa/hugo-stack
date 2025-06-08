---
title: "[API] OpenAPI & Swagger 簡介"
categories: ["Programming"]
date: 2020-10-05 14:15:00
slug: "openapi-introduction"
---

## OpenAPI

OpenAPI  是一套  API  規範（OpenAPI Specification ，OAS），用於定義  RESTful API。API  規範使用  YAML  或  JSON  編寫，對於人類和機器來說皆易於閱讀。

<!--more-->

OpenAPI 簡單來說就是  `API`，一般是指一開始是封閉的系統，比如最開始的  Twitter、Google  或者  Facebook。突然有一天，他們開放了！公佈了實現某些功能的  API，來獲得他們內部的數據、執行特定操作。這個時候，這樣的  API，  我們就可稱為  "Open" API。

## Swagger

Swagger  是一套以  OpenAPI  規範構建的開源工具，可以幫助你設計、構建、記錄和使用  REST API。主要的  Swagger  工具包括：

- **Swagger Editor**：基於瀏覽器的編輯器，你可以在其中編寫  OpenAPI  規範。

- **Swagger UI**：將  OpenAPI  規範呈現為交互式的  API  文檔，使得用戶可以直接在瀏覽器中調用  API。

- **Swagger Codegen**：一個開源的代碼產生器，根據定義好的 RESTful API 文件產生 server stubs 及 client SDKs。

## 為什麼要使用  Open API

### 標準化

遵循  OAS  規範的  RESTful API  定義，可使  API  互通介面不受程式語言限定。且統一採用  API  進行跨系統呼叫，使得內部系統的介接標準一致，即便未來得換掉某一套系統，API  串接仍保持一致，其他系統不用特別修改寫法就能繼續呼叫。

### 節省開發時間及成本

使用規範好的框架來設計  API。再開發者不需要從頭了解  API  就能開始串接，可以加快開發時間。

## Reference

- https://kknews.cc/code/nrgp5aq.html
- https://www.ithome.com.tw/news/133682
- https://www.zhihu.com/question/20225153
