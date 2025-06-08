---
title: "[Python] Install Python3.9 on CentOS7"
tags:
  - Python
categories:
  - Programming
date: 2021-01-15 15:51:00
slug: install-python39-on-centos7
---

## step1

下載依賴工具以及安裝包

```sh
$ yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel wget
$ wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz
$ tar zxvf Python-3.9.1.tgz
```

<!--more-->

## step2

編譯 Python

```sh
$ cd Python-3.9.1
# 檢測並產生 Makefile，且指定安裝目錄
$ ./configure prefix=/usr/local/python3
# 開始編譯
$ make && make install
```

## step3

因為想直接用 `python` command 代表使用 python3，所以新增別名

```sh
$ vi ~/.bashrc
```

</br>

```
alias python='/usr/local/python3/bin/python3'
alias pip='/usr/local/python3/bin/pip3'
```

</br>

```sh
$ source ~/.bashrc
```

## step4

確認安裝成功

```sh
$ python -V
$ pip -V
```

## 補充 (virtualenv)

如果想保留系統預設的 python 2.7，則不要新增上面第三步的別名，相反的加上 python3 執行路徑的環境變數。

```sh
$ vi ~/.bashrc
```

</br>

```
export python3='/usr/local/python3/bin'
```

</br>

```sh
$ source ~/.bashrc
```

為了區隔兩個 python 的版本，可以使用 virtualenv 隔離不同的開發環境。

```sh
$ pip3 install virtualenv
$ python3 -m virtualenv project1
$ cd project1
$ source bin/activate
$ python --version
Python 3.9.1
```

</br>

{{< notice info >}}
環境變數設定小補充
/etc/profile --> 對所有用戶永久生效
~/.bashrc --> 對單一用戶永久生效，當你"啟動" shell 時執行
~/.bash_profile --> 對單一用戶永久生效，當你"登入" shell 時執行
export xxx = xxx -->直接運行 export 命令定義變量，只對當前 shell 有效，關閉 shell 終端後失效。
{{< /notice >}}

## Refernce

- https://blog.jiebu-lang.com/centos-7-install-python-3-7/
- https://liqiang.io/post/install-python3-8-in-centos-973bdb81
- https://blog.csdn.net/qq_36758461/article/details/103841798
