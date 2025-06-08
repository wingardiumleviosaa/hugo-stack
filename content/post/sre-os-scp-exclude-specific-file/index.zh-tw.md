---
title: "[Linux] SCP 傳送排除特定檔案或資料夾 (如 .git)"
categories: ["SRE"]
tags: ["Linux"]
date: 2021-11-08 11:03:00
slug: linux-scp-exclude-files
---

<!--more-->

Scp all files and folders to the remote server without copying .git & other dot files/folders:

```
cd sourceFolder
scp -r [!.]* root@remoteServer:/root/targetForder
```

The command means transfer all the files/folders `*` under current directory except `[!]` the files'/folders' named starting with `.`
