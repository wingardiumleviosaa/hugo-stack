---
title: "[Golang] Method"
tags:
  - Golang
categories:
  - Programming
date: 2019-12-29 05:06:00
slug: golang-method
---

{{< notice info >}}
Golang is all about TYPE!
{{< /notice >}}

## 定義

method 是一個額外帶有 receiver(接收器)的 fuction，receiver 為某 type 的變量，而 type 的型態可以是 struct 或是 non-struct

<!--more-->

語法如下

```
func (r receiverType) identifier(parameter) returnTypes {}
```

範例如下

```go
	type student struct {
		name string
		school int
	}

	func (s student) greet() string {
		return fmt.Sprintf("Hi, I'm %v from %v.", s.name, s.school)
	}
```

## 呼叫 method

type 的變量則透過 `.` 呼叫 method：

```go {linenostart=9}

func main() {
		s1 := student{"Alice", 112}
		s2 := student{"Bob", 118}    //s1 & s2為student type 的變數

		fmt.Println(s1.greet())
		fmt.Println(s2.greet())    //透過variable.method呼叫
	}
```

## Pointer v.s. Value as a receiver

method 中的 receiver 可以是值，也可以是 pointer，若為 pointer，則可以直接修改 receiver 中的內容。  
:loudspeaker: 使用 pointer 傳遞的時機 :loudspeaker:

{{< notice warning >}}

1.  用於欲改變 receiver 變數的值時
2.  struct 本身很大，複製代價高時
3.  當 struct 的 method 有一個 receiver 為 pointer 時，則其餘的 method 的 receiver 也必須使用 pointer
    {{< /notice >}}

以下範例展現傳值(value)及傳址(pointer)的差異，請留意此範例打破了上面的第三點原則，僅是為了要顯示兩者的異同：

```go
		package main

		import "fmt"

		type student struct {
			name   string
			school int
		}

		func (s student) transByValue(x int) string {
			s.school = x
			return fmt.Sprintf("Hi, I'm %v from %v.", s.name, s.school)
		}

		func (s *student) transByInference(x int) string {
			s.school = x
			return fmt.Sprintf("Hi, I'm %v from %v.", s.name, s.school)
		}

		func main() {
			s1 := student{"Alice", 112}

			fmt.Println(s1.transByValue(118))
			fmt.Println(s1.school)

			fmt.Println((&s1).transByValue(117))
			fmt.Println(s1.school)

			fmt.Println(s1.transByInference(113))
			fmt.Println(s1.school)

			fmt.Println((&s1).transByInference(114))
			fmt.Println(s1.school)

		}

```

執行結果

```
Hi, I'm Alice from 118.
112
Hi, I'm Alice from 117.
112
Hi, I'm Alice from 113.
113
Hi, I'm Alice from 114.
114
```

由上述範例中，可以發現到：  
{{< notice warning >}}
:mag_right: 當使用 value 作為 receiver 傳遞時，函數僅 copy 了值，並無改變到變數本身  
:mag_right: 當使用 pointer 作為 receiver 傳遞時，是傳遞整個變數，故會改到變數本身
{{< /notice >}}
