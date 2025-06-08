---
title: >-
  [WSL2] Ubuntu 22.04 LTS nsenter: cannot open /proc/xxx/ns/time: No such file
  or directory
tags:
  - WSL
categories:
  - SRE
date: 2022-09-06 21:17:00
slug: wsl2-ubuntu-nsenter-cannot-open-proc-no-such-file-or-directory
---

## 問題

在使用 https://github.com/DamionGans/ubuntu-wsl2-systemd-script.git 安裝 systemd 後，重啟 wsl，再開 terminal 時會遇到 `cannot open /proc/xxx/ns/time: No such file or directory` 的錯誤。

<!--more-->

## 解決方法

使用 powershell 開啟 wsl

```
wsl -d Ubuntu-22.04 -e bash --norc
```

修改 `enter-systemd-namespace` 檔案

```
sudo vi /usr/sbin/enter-systemd-namespace
```

將下面幾行取代原先的區塊

```sh
USER_HOME="$(getent passwd | awk -F: '$1=="'"$SUDO_USER"'" {print $6}')"
if [ -n "$SYSTEMD_PID" ] && [ "$SYSTEMD_PID" != "1" ]; then
    if [ -n "$1" ] && [ "$1" != "bash --login" ] && [ "$1" != "/bin/bash --login" ]; then
        exec /usr/bin/nsenter -t "$SYSTEMD_PID" -m -p \
            /usr/bin/sudo -H -u "$SUDO_USER" \
            /bin/bash -c 'set -a; [ -f "$HOME/.systemd-env" ] && source "$HOME/.systemd-env"; set +a; exec bash -c '"$(printf "%q" "$@")"
    else
        exec /usr/bin/nsenter -t "$SYSTEMD_PID" -m -p \
            /bin/login -p -f "$SUDO_USER" \
            $([ -f "$USER_HOME/.systemd-env" ] && /bin/cat "$USER_HOME/.systemd-env" | xargs printf ' %q')
    fi
    echo "Existential crisis"
    exit 1
fi
```

退出 powershell terminal 之後再重新使用 wsl terminal 就可以順利進入系統。

## 補充

在每次重開機後，WSL 的 docker 都不會自動被叫起來

![](https://imgur.com/ACyTK2P.png)

處理方式為：
開啟 powershell，關閉對應 wsl

```
wsl -t Ubuntu-22.04
```

</br>

![](https://imgur.com/e4UCZ6K.png)

進入 Docker Desktop，取消對應的 wsl integration 勾選，按 `Apply & Restart`

![](https://imgur.com/Z3EfEW0.png)

重啟後再重複勾選 wsl integration，並重啟

![](https://imgur.com/thgGVtG.png)

回到 powershell，重新啟動 wsl `wsl -d Ubuntu-22.04` 後，就可以使用 docker 了。

![](https://imgur.com/XKjredF.png)

## Reference

- https://github.com/DamionGans/ubuntu-wsl2-systemd-script/issues/76#issuecomment-1122717349
- https://github.com/DamionGans/ubuntu-wsl2-systemd-script/issues/36#issuecomment-732090101
