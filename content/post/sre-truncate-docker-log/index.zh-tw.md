---
title: "[Docker] 限制 log 大小以避免硬碟爆掉"
tags:
  - Docker
categories:
  - SRE
date: 2020-08-14 11:15:00
slug: docker-truncate-docker-log
---

默認情況下，Docker 會抓所有 container 的標準輸出和標準錯誤 (stdout & stderr)，並將其寫入 `/var/lib/docker/containers/[container-id]/[container-id]-json.log` 的 json 文件中。

<!--more-->

當 container 運作時間愈長，Log 檔案會隨之變大，進而導致機器硬碟空間被 Log 佔據。因此需限制 Log 的檔案大小，以避免硬碟被塞爆。例如以下這個 log 就佔了 1G 多的磁碟空間：

![](https://imgur.com/z1p61Ox.png)

## 手動清除日誌

可以使用下列幾種方法清除該 log 檔案內容：

### redirection operator

```
> <log file>
```

- `>` 重新導向運算子，因檔案已經存在，所以重新導向後會清空原有的內容。

### truncate

```
truncate --size 0 <log file>
```

- truncate : 將文件縮小或放大到指定大小的命令。

- --size : 指定大小的參數。

### /dev/null

```
cat /dev/null > <log file>
```

- /dev/null：是在類 Unix 操作系統上稱為空設備的特殊設備，當從中讀取數據時不會有任何數據，在寫入時也不存儲任何數據。

![](https://imgur.com/ZJk0xmk.png)

上圖使用第一種清除方法，可以看到 1G 的空間被釋放！

但每次都手動清除是件很麻煩的事，可以考慮使用 `cronjob` 設定排程，或是直接設定 docker log 的容量上限。

## 設定自動日誌滾動

docker 默認的日誌驅動程式設定可以在 `/etc/docker/daemon.json` 中定義，若無此文件，則需額外新增，設定值可參考下列內容：

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```

</br>

| 選項     | 描述                                                                                                                               |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| max-size | 滾動前日誌的最大大小。一個正整數加上一個代表測量單位（k，m 或 g）的修飾符。默認為 -1（代表無限制）                                 |
| max-file | 可以存在的最大日誌文件數量。如果滾動日誌會創建多余文件，則會刪除最舊的文件。只有在設置了 max-size 時才有效。一個正整數。默認為 1。 |

執行以下命令來重新加載 docker daemon。<font style="background:MistyRose">新的配置將在重新啟動後適用於所有<span class="dotunderletter">**新**建立</span>的容器，現有容器即使重啟 Docker 也不會使用新的日誌記錄配置。</font>

```
systemctl daemon-reload

systemctl restart docker
```

## 個別為單一容器設置日誌滾動

若不想全局配置，也可以在個別容器啟動時，在 command 中加入參數改動。

### docker run

```bash
docker run \
    --log-driver json-file \
    --log-opt max-size=10m \
    --log-opt max-file=10
    ubuntu:18.04
```

### docker-compose

```yaml
version: "3.2"
services:
  nginx:
    image: "nginx:latest"
    ports:
      - "80:80"
    logging:
      driver: "json-file"
      options:
        max-size: "1k"
        max-file: "3"
```

## Reference

- https://www.rdaemon.com/truncate-files-using-command-line-tools
- https://blog.csdn.net/kikajack/article/details/79575659
- https://blog.boatswain.io/zh/post/docker-container-log-rotation/
