---
title: "[Golang] Function"
tags:
  - Golang
categories:
  - Programming
date: 2020-01-04 20:02:00
slug: golang-function
---

## Introduction

在 Go 裡，function 是一等公民，它可以作為另一 function 中的參數傳遞、可以將它指派給一變數、也可以在另一個 function 中當作 return 值。

函式可以具名也可以匿名(anonynous function)；

<!--more-->

一般函式  
`func (r receiverType) identifier(arguments argumentTypes) returnType{}`

function 中的參數以及返回值可以有零個或多個：

```go
\\傳入零個argument, 無回傳值
func f1(){
	fmt.Println("hello!")
}

\\傳入兩個arguments, 回傳兩個return values
func f2(x, y int) (int, int) {
    return x + y, x - y
}

\\傳入未知個數的多個arguments
func f3(x ...int) {
	for _, v := range x{
    fmt.Println(x)
    }
}
```

匿名函式為沒有名字的函式  
`func(arguments) returnType{}`

以下將針對 function 作為一等公民的實踐做闡述。

## Callback 回呼函式

<table><tr><td bgcolor=AliceBlue>
<font size= 3><b>定義</b></font></br>
把function A當作參數傳進另一個function B內，故在執行B的時候會callback回去參考A。
</td></tr></table>

以下範例先定義一個用於加總的函式 sum，再定義另一個用於計算奇數總和的函式 odd_sum，在 odd_sum 中丟入 sum 當作參數去計算總合，實現 callback。

```go
package main

import (
	"fmt"
)

func main() {
	i := []int{1, 2, 3, 4, 5, 6, 7}
	oddsum := odd_sum(sum, i...) //把sum函數當參數傳入
	fmt.Println("The sum of the odd number in the list I is", oddsum)
}

func sum(x ...int) int {
	total := 0
	for _, v := range x {
		total += v
	}
	return total
}


//使用func(x ...int) int 當函式傳入, 函式變數為f
func odd_sum(f func(x ...int) int, y ...int) int {
	oddlist := []int{}
	for _, v := range y {
		if v%2 == 1 {
			oddlist = append(oddlist, v)
		}
	}
	return f(oddlist...)
}
```

## Nested function 內嵌函式

<table><tr><td bgcolor=AliceBlue>
<font size= 3><b>定義</b></font></br>
在 Go 中，不允許在function A中又宣告另一個function B，但可以把匿名的function B指定給某變數並放在function A中。
</td></tr></table>

在 func main 宣告變數 f 為匿名函式：

```go
package main
import "fmt"

func main(){
	x := 0
	f := func() int{
			x++
			return x
		}
	fmt.Println(f())
}
```

## closure 閉包

<table><tr><td bgcolor=AliceBlue>
<font size= 3><b>定義</b></font></br>
當<b>匿名函數中使用了<span class="dotunderletter">外部變數</span>x</b>，此時這個匿名函式形成一個閉包。所以閉包可以視為匿名函式的特殊案例之一。
</td></tr></table>

乘上範例，當重複呼叫 f()時，會發現 x 的值存留在函式中：

```diff=10
	fmt.Println(f())
+	fmt.Println(f())
+	fmt.Println(f())
}
```

執行結果

```
1
2
3
```

為了更實踐閉包<font color=DarkBlue>**數據隔離**</font>的優勢，我們將 x 與匿名函數移出 main function 外獨立成為一個函式

其中我們<u>使用匿名函式 func() int{...}作為 function bar 的回傳值</u>

```go
package main
import "fmt"

func main(){
	f := foo()
    fmt.Println(f())
	fmt.Println(f())
    fmt.Println(f())
}

func foo() func() int{
	x := 0
	return func() int {
		x++
		return x
	}

```

執行結果

```
1
2
3
```

此時即使我們執行完 foo()，變數 x 仍然存在在閉包中，外部無法訪問汙染。
