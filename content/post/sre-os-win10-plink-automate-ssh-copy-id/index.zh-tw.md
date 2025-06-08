---
title: "[Windows] 用 plink 自動化 ssh-copy-id 到指定機器"
tags:
  - Windows
categories:
  - SRE
date: 2022-06-13 15:13:00
slug: win10-plink-automate-ssh-copy-id
---

<!--more-->

研究怎麼不透過人為輸入密碼，將本地主機上的公鑰上傳到遠端主機的 authorized_keys 文件中，以實現 SSH 金鑰驗證。

```sh
cat id_rsa.pub | plink -ssh root@10.37.39.69 -pwfile pw.txt "cat >> ~/.ssh/authorized_keys"
```

</br>

{{<notice info>}}
Plink 是由 PuTTY 套件提供的命令行工具，多被使用在自動執行的場景應用，可在自動化部署執行 SSH 的指令。
{{</notice>}}
