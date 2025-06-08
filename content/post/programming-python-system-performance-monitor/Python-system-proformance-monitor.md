---
title: "[Python] 系統效能監測"
tags:
  - Python
categories:
  - Programming
date: 2021-01-15 15:54:00
slug: monitor-system-perf-by-python
---

監測的指標有 CPU 使用率&頻率、已使用的記憶體、磁碟讀寫 bytes 數、網路收發 bytes 數。會將每秒讀取到的數值存為 csv 檔。

## Prerequisite

- Python3.5+
- psutil

<!--more-->

## code

```python
import psutil
import time
from collections import namedtuple

################################################################################
class SystemMonitor(object):
    running = True
    CPU = namedtuple('cpu', ['percentage', 'freq'])
    NET = namedtuple('net', ['rx', 'tx'])
    DISK = namedtuple('disk', ['read', 'write'])
    SysMon = namedtuple('sysmon', ['timestamp', 'cpu', 'vm', 'disk', 'net'])

    def __init__(self, csvfile='./sysmon.csv', interval=1.0):
        self.interval = interval
        self.csvfile = csvfile

    def run(self):
        self.info()
        with open(self.csvfile, 'w') as fp:
            fp.write('timestamp,percentage,freq,vm,disk_rd,disk_wr,net_rx,'
                     'net_tx\n')

        while self.running:
            time.sleep(self.interval)
            info = self.info()
            #print(info.timestamp)
            info = [
                info.timestamp,
                info.cpu.percentage,
                info.cpu.freq,
                info.vm,
                info.disk.read,
                info.disk.write,
                info.net.rx,
                info.net.tx,
            ]
            info = ','.join([str(x) for x in info]) + '\n'
            with open(self.csvfile, 'a+') as fp:
                fp.write(info)

    def info(self):
        ts = int(time.time())
        cpu = self.CPU(psutil.cpu_percent(), psutil.cpu_freq().current)
        vm = psutil.virtual_memory().used
        diskio = psutil.disk_io_counters()
        netio = psutil.net_io_counters()

        result = self.SysMon(
            ts, cpu, vm,
            self.DISK(diskio.read_bytes, diskio.write_bytes),
            self.NET(netio.bytes_recv, netio.bytes_sent))
        return result

################################################################################
if __name__ == '__main__':
    m = SystemMonitor()
    m.run()
```
