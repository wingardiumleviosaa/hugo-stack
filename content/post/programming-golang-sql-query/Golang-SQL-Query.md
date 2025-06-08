---
title: "[Golang] 查詢 SQL DB 的幾種匯出方式"
tags:
  - Golang
categories:
  - Programming
date: 2022-04-19 21:19:00
slug: golang-sql-query
---

簡易紀錄一下 sql query 回傳(印出)查詢結果的三種不同方法，三種方式回傳的格式都差不多，只是型態可能不一樣。本文使用的 database 是 Oracle。

<!--more-->

## 連線

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/godror/godror"
)

const (
	username = "ula"
	password = "123456"
	host     = "10.90.1.207"
	port     = 1521
	sid      = "emesHY"
)

func main() {

	oralInfo := fmt.Sprintf("%s/%s@%s:%d/%s", username, password, host, port, sid)
	db, err := sql.Open("godror", oralInfo)

	err = db.Ping()
	if err != nil {
		fmt.Println("cant connect")
		db.Close()
	}

	sqlComand := fmt.Sprintf("SELECT DISTINCT SERIAL_NUMBER,PRE_KPSN,THIS_KPSN,DATECODE,LOTCODE from EMESC.VP_ASSY_SUMMARY_SMT where serial_number = 'TBCBB2039913'")

	rows, err := db.Query(sqlComand)
	if err != nil {
		fmt.Println("Query Database Failed!")
	}
	defer rows.Close()

	// 插入以下其中一種作法
}
```

## 作法一

```go
columns, err := rows.Columns()
if err != nil {
	fmt.Println(err)
}
count := len(columns)
tableData := make([]map[string]interface{}, 0)
values := make([]interface{}, count)
valuePtrs := make([]interface{}, count)
for rows.Next() {
	for i := 0; i < count; i++ {
		valuePtrs[i] = &values[i]
	}
	rows.Scan(valuePtrs...)
	entry := make(map[string]interface{})
	for i, col := range columns {
		var v interface{}
		val := values[i]
		b, ok := val.([]byte)
		if ok {
			v = string(b)
		} else {
			v = val
		}
		entry[col] = v
	}
	tableData = append(tableData, entry)
}
jsonData, err := json.Marshal(tableData)
if err != nil {
	fmt.Println(err)
}
// fmt.Println(string(jsonData))

var result []map[string]interface{}

if err := json.Unmarshal([]byte(jsonData), &result); err != nil {
	fmt.Println("Can't parse the json string")
}

fmt.Println(result)
```

## 作法二

```go
type Kpsn struct {
	SN   string `json:"Serial_Number"`
	Pre  string `json:"Pre_KPSN"`
	This string `json:"This_KPSN"`
	DC   string `json:"Datecode"`
	LC   string `json:"Lotcode"`
}

var kpsns []*Kpsn

for rows.Next() {
	k := new(Kpsn)
	rows.Scan(&k.SN, &k.Pre, &k.This, &k.DC, &k.LC)
	kpsns = append(kpsns, k)
}
jsonStr, _ := json.Marshal(&kpsns)

var result []map[string]interface{}
// var result []Kpsn

if err := json.Unmarshal([]byte(jsonStr), &result); err != nil {
	fmt.Println("Can't parse the json string")
}

fmt.Println(result)

```

## 作法三

```go
var sn, pre, this, dc, lc string
var result []map[string]interface{}

for rows.Next() {
	rows.Scan(&sn, &pre, &this, &dc, &lc)
	var tmp map[string]interface{}
	tmp = make(map[string]interface{})
	tmp["SERIAL_NUMBER"] = sn
	tmp["PRE_KPSN"] = pre
	tmp["THIS_KPSN"] = this
	tmp["DATECODE"] = dc
	tmp["LOTCODE"] = lc
	result = append(result, tmp)
}
fmt.Println(result)
```

## Reference

- https://stackoverflow.com/a/29164115/13318115
