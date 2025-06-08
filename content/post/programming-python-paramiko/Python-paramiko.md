---
title: "[Python] Paramiko"
tags:
  - Python
categories:
  - Programming
date: 2020-09-30 14:26:00
slug: python-paramiko
---

## 簡介

paramiko  是一個使用  SSH2  遠端控制的模組，可以對遠端服務器進行命令或文件操作。有兩個核心組件：

<!--more-->

- SSHClient：它的作用類似於  Linux  的  SSH  命令，是對  SSH  會話  (Session) \*  的一個類的封裝，這個類封裝了傳輸(Transport) \*、通道(Channel)\*  及  SFTPClient  建立的方法  (open_sftp)。
- SFTPClient：它的作用類似  Linux  的  SFTP  命令，是對  SFTP  客戶端的一個類的封裝。主要是實現對遠端文件的操作，上傳、下載、修改文件權限等操作。

<table><tr><td bgcolor=#FAFAFA>
* Transport：是一種加密的會話 (session)，使用時會同步創建一個加密的 Channel（即為一個 socket）。<br/>
* Session：client 和 server 保持連接的對象。</span>
</td></tr></table>

## SSHClient()：遠端  SSH  登入

```python
import paramiko
   # new a SSHClient instant
   client = paramiko.SSHClient()
 
   # 使用自動添加策略，保存伺服器的 hostname 和金鑰資訊
   client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
 
   # connect the server through ssh using password to login
   client.connect(hostname='10.1.5.1', port=22, username='root', password='0000')
   # 或是使用金鑰登入
   # private = paramiko.RSAKey.from_private_key_file('/home/root/.ssh/id_rsa')
   # client.connect(hostname='10.1.5.1',port=22,username='root',pkey=private)
```

## exec_command()  以及  invoke_shell()  的用法與差別

- exec_command()  函數使用  SSH exec  通道，會在執行命令後關閉通道，不同通道間不保證保留互相狀態(例如工作目錄或是變數)。
- invoke_shell()  函數使用  SSH shell  通道，會實現交互式  shell  會話，在  stream  關閉前會在同一個通道中執行命令。

### exec_command()

```python {linenostart=13}
   # open a channel and execute the command
   stdin, stdout, stderr = client.exec_command('df -h')  
   # stdout 為正確輸出，stderr 為錯誤輸出，同時只有一個變量有值
 
   # print out the result
   print(stdout.read().decode('utf-8'))
 
   # close SSHClient
   client.close()
```

### invoke_shell()

```python {linenostart=13}
    command='dh -h'
    chan=ssh.invoke_shell()
    chan.send(command+'\n')
    # \n 是執行命令的意思，沒有 \n 不會執行
    time.sleep(1) # 等待執行
    res=chan.recv(1024) # 接受返回訊息
    chan.close()
```

## SSHClient  封装  Transport

```py
import paramiko
 
   # create a channel
   transport = paramiko.Transport(('hostname', 22))
   transport.connect(username='root', password='0000')
 
   ssh = paramiko.SSHClient()
   ssh._transport = transport
 
   stdin, stdout, stderr = ssh.exec_command('df -h')
   print(stdout.read().decode('utf-8'))
 
   transport.close()
```

## Reference

- https://stackoverflow.com/questions/6770206/what-is-the-difference-between-the-shell-channel-and-the-exec-channel-in-jsc
- https://www.cnblogs.com/xiao-apple36/p/9144092.html
