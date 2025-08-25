---
title: "【EC200A 学习笔记】-- ASR1803S+AIC8800DW+SZ18201 简单测速结果"
date: 2024-11-08T00:49:37+08:00
lastmod: 2024-11-08T00:49:37+08:00
author: ["hacper"]
tags:
    - ASR1803S
    - EC200A
    - RTOS
    - AIC8800DW
    - SZ18201
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "ASR1803S+AIC8800DW+SZ18201 简单测速的验证结果。" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true
---

硬件环境：ASR1803S+AIC8800DW+SZ18201，手机连接 ASR1803S 热点，PC 端通过网线连接 ASR1803S，简单测速验证结果。

```bash
# PC 端作为 server， 手机端作为 client
❯ .\iperf3.exe -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 192.168.0.200, port 39734
[  5] local 192.168.0.100 port 5201 connected to 192.168.0.200 port 39744
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  4.07 MBytes  34.1 Mbits/sec
[  5]   1.00-2.00   sec  4.22 MBytes  35.3 Mbits/sec
[  5]   2.00-3.00   sec  4.18 MBytes  35.1 Mbits/sec
[  5]   3.00-4.00   sec  4.35 MBytes  36.5 Mbits/sec
[  5]   4.00-5.00   sec  4.29 MBytes  35.9 Mbits/sec
[  5]   5.00-6.00   sec  4.31 MBytes  36.3 Mbits/sec
[  5]   6.00-7.00   sec  4.26 MBytes  35.7 Mbits/sec
[  5]   7.00-8.00   sec  4.09 MBytes  34.3 Mbits/sec
[  5]   8.00-9.00   sec  4.32 MBytes  36.2 Mbits/sec
[  5]   9.00-10.00  sec  4.42 MBytes  37.0 Mbits/sec
[  5]  10.00-10.06  sec   218 KBytes  30.2 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-10.06  sec  42.7 MBytes  35.6 Mbits/sec                  receiver
-----------------------------------------------------------
```

```bash
# PC 端作为 client， 手机端作为 server
❯ .\iperf3.exe -c 192.168.0.200
Connecting to host 192.168.0.200, port 5201
[  4] local 192.168.0.100 port 63669 connected to 192.168.0.200 port 5201
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-1.01   sec  2.12 MBytes  17.7 Mbits/sec
[  4]   1.01-2.01   sec  3.88 MBytes  32.6 Mbits/sec
[  4]   2.01-3.01   sec  4.50 MBytes  37.7 Mbits/sec
[  4]   3.01-4.01   sec  4.75 MBytes  39.7 Mbits/sec
[  4]   4.01-5.00   sec  5.00 MBytes  42.4 Mbits/sec
[  4]   5.00-6.01   sec  4.75 MBytes  39.5 Mbits/sec
[  4]   6.01-7.01   sec  4.75 MBytes  39.9 Mbits/sec
[  4]   7.01-8.01   sec  6.12 MBytes  51.4 Mbits/sec
[  4]   8.01-9.00   sec  6.00 MBytes  50.8 Mbits/sec
[  4]   9.00-10.00  sec  5.75 MBytes  48.2 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  4]   0.00-10.00  sec  47.6 MBytes  39.9 Mbits/sec                  sender
[  4]   0.00-10.00  sec  47.6 MBytes  39.9 Mbits/sec                  receiver

iperf Done.
```

然后实网测速结果：
![](https://github.com/hacperme/picx-images-hosting/raw/master/20241108/cdfaaa201d0d42722f2c9c3aa3ad254.6m3wz3nffy.webp)
