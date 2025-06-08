---
title: "[Golang] 使用 winrm 下多行指令操作遠端機器"
tags:
  - Golang
categories:
  - Programming
date: 2023-01-28 08:19:00
slug: golang-winrm-multiple-command-sample-code
---

## TL; DR

使用 [masterzen/winrm](https://github.com/masterzen/winrm) 套件透過 winrm protocal 遠端連線至指定機器並下 powershell 指令操作。

<!--more-->

## 程式碼

```go
// 直接寫成一個 function~
func WinExec() {
	params := winrm.DefaultParameters

	endpoint := winrm.NewEndpoint("10.37.36.79", 5985, false, false, nil, nil, nil, 0)
	client, err := winrm.NewClientWithParameters(endpoint, "Administrator", "Wistron@2017", params)
	if err != nil {
		panic(err)
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	shell, err := client.CreateShell()
	if err != nil {
		panic(err)
	}
	defer shell.Close()

	// 可以在這邊下多行指令
	script := fmt.Sprintf(`
	ipconfig
	`)

	cmd, err := shell.ExecuteWithContext(ctx, winrm.Powershell(script))
	// _, err = client.RunWithContext(ctx, "echo username", os.Stdout, os.Stderr)

	if err != nil {
		panic(err)
	}

	defer cmd.Close()

	var wg sync.WaitGroup
	copyFunc := func(w io.Writer, r io.Reader) {
		defer wg.Done()
		io.Copy(w, r)
	}

	wg.Add(2)
	go copyFunc(os.Stdout, cmd.Stdout)
	go copyFunc(os.Stderr, cmd.Stderr)

	cmd.Wait()
	wg.Wait()

	if cmd.ExitCode() != 0 {
		panic(cmd.ExitCode())
	}
}
```
