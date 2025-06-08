---
title: VSCode 免密碼 SSH 到遠端機器
categories: ["SRE"]
date: 2021-10-08 17:25:00
slug: vscode-remote
---

<!--more-->

## VS Code 安裝插件

![](https://imgur.com/DpEan3E.png)

安裝完成後我們就可以看到在左側新增了一個新的 Remote 欄位

![](https://imgur.com/ToGz1Ho.png)

到 `Settings > Extensions >Remote - SSH` 中將 `Remote.SSH: Show Login Terminal` 勾選

![](https://imgur.com/Ox8ScCa.png)

## Windows 設定安裝 OpenSSH

使用 Windows Server 2019 和 Windows 10 裝置上的 Windows 設定，安裝 OpenSSH 元件。

若要安裝 OpenSSH 元件：

1. 開啟  **設定**，選取 [應用程式] **> 應用程式 & 功能**，然後選取 [**選用功能**]。
2. 掃描清單，查看是否已安裝 OpenSSH。 如果沒有，請在頁面頂端選取 [ **新增功能**]，然後：
   - 尋找  **OpenSSH 用戶端**，然後按一下 [**安裝**]。
   - 尋找  **OpenSSH 伺服器**，然後按一下 [**安裝**]。

安裝程式完成後，請返回  **應用程式 > 應用程式 & 功能**  和  **選用功能** ，應該會看到已列出 OpenSSH。

## SSH Key 設定

Remote-SSH extension 提供我們使用 ssh key 的方式進行連接，可以不用輸密碼更方便的連接。方式是將本地端的公鑰（id_rsa.pub）存到遠端的 `authorized_keys` 檔案中，然後在本地端的 config 檔中設定連線資訊。

1. 在本地機器上使用 `ssh-keygen` 產生 key pair
2. 將本地公開金鑰加入到遠端機器上的 `authorized_keys` 檔案中，若遠端機器無該檔案，則 `touch ~/.ssh/authorized_keys`，並修改權限 `chmod 644 ~/.ssh/authorized_keys`，最後將本地端的 id_rsa.pub 內容複製到檔案中。
3. 修改本地端機器的家目錄下的 .ssh 中的 `C:\Users\%USER%\.ssh\config` 檔
   ```
   Host remoteLinux              #填寫別名例如 LabSever
   HostName 10.1.5.130           #主機名稱或是ip位置
   User root                     #登入的使用者名稱
   IdentityFile ~/.ssh/id_rsa    #指定連線的私鑰
   ```
4. 測試是否可 ssh 登入，開啟 CMD 執行指令  `ssh 登入帳戶@ServerIP`，即可免密碼直接登入 Linux Server，如果金鑰名稱不為預設 id_rsa，則使用 `-i` 參數指定金鑰 `ssh 登入帳戶@ServerIP -i C:\Users\%USER%\.ssh\mykey`。

## SSH Key 原理

透過[公開金鑰加密](https://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86) (Public-key cryptography) 或稱「非對稱金鑰加密」的兩把加解密鑰匙 Public Key (公鑰) 和 Private Key (私鑰)，取代 Client 使用  [SSH](https://zh.wikipedia.org/wiki/Secure_Shell) (Secure Shell，安全殼協議) 協定連結 Server 登入時必須輸入驗證密碼的動作：

1. Public Key：流通於公開網路上，在各伺服器上公鑰集中保存的檔案為 `authorized_keys`，必須依據 /etc/ssh/sshd_config 內的 AuthorizedKeysFile 定義來設定。
2. Private Key：存放於各伺服器的本地端，用來解密公開給不同來源端的 Public Key，不可外流。

## 權限

雖然 ssh-keygen 預設的權限就都是正確的了，但還是有必要了解一些規則，因為只要一個設定有誤，就可能會被判定為危險，而造成 Public Key 和 Private Key 無法順利比對：
目錄或檔案|許可權
---------|-----
~/.ssh/|700 (drwx------)
~/.ssh/id_rsa (Private Key)|600 (-rw-------)
~/.ssh/authorized_keys & ~/.ssh/id_rsa.pub (Public Key)|600 (-rw-------)。預設為 600，可以將許可權變更為 640 或 644，使公開金鑰變成可讀取。

## Reference

- [https://hackmd.io/@brick9450/vscode-remote](https://hackmd.io/@brick9450/vscode-remote)
- [https://www.footmark.info/linux/centos/windows-ssh-nopassword-linux/](https://www.footmark.info/linux/centos/windows-ssh-nopassword-linux/)
