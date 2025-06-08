---
title: MongoDB 一次為舊有資料加上新增欄位
tags:
  - MongoDB
categories:
  - DataEngineering
  - Database
date: 2022-05-19 11:15:00
slug: mongodb-update-new-field-to-old-document
---

<!--more-->

```sh
db.collection1.update({"cameras":{$exists:false}}, {$set:{"cameras":{}}},false,true)
```

最後兩個 false, true 參數個別代表：
- Upsert: If set to true, creates a new document when no document matches the query criteria.
- Multi: If set to true, updates multiple documents that meet the query criteria. If set to false, updates one document.