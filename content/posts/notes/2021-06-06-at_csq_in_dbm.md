---
id: 1948
title: 'AT+CSQ 与 dBm 的那些事'
date: '2021-06-06T22:34:54+08:00'
author: hacper
excerpt: 'AT+CSQ 是一条查询设备信号质量的 AT 指令，介绍CSQ与dBm的关系。'
layout: post
guid: 'https://hacperme.com/?p=1948'
permalink: /2021/06/06/at_csq_in_dbm/
views:
    - '1731'
categories:
    - 笔记
tags:
    - AT
    - CSQ
    - dBm
---

AT+CSQ 是一条查询设备信号质量的 AT 指令。下面是一个查询信号质量的示例：

```shell
AT+CSQ=?

+CSQ: (0-31,99),(0-7,99)

OK

AT+CSQ

+CSQ: 28,99

OK

```

AT指令响应+CSQ: 28,99 中的信号等级28表示 rssi -57dBm，其转换公式如下：

```
dBm = -113 + N * 2 (where N is the returned value)
```

![csq_in_dBm](https://cdn.jsdelivr.net/gh/hacperme/picx_hosting@master/20210507/csq_in_dBm.7aknpv7dfrk0.png)

CSQ与dBm的对应关系如下表：

| <th>**Value**</th> | <th>**RSSI dBm**</th> | <th>**Condition**</th> |
|--------------------|-----------------------|------------------------|
| 0 | -113 | Marginal |
| 1 | -112 | Marginal |
| 2 | -109 | Marginal |
| 3 | -107 | Marginal |
| 4 | -105 | Marginal |
| 5 | -103 | Marginal |
| 6 | -101 | Marginal |
| 7 | -99 | Marginal |
| 8 | -97 | Marginal |
| 9 | -95 | Marginal |
| 10 | -93 | OK |
| 11 | -91 | OK |
| 12 | -89 | OK |
| 13 | -87 | OK |
| 14 | -85 | OK |
| 15 | -83 | Good |
| 16 | -81 | Good |
| 17 | -79 | Good |
| 18 | -77 | Good |
| 19 | -75 | Good |
| 20 | -73 | Excellent |
| 21 | -71 | Excellent |
| 22 | -69 | Excellent |
| 23 | -67 | Excellent |
| 24 | -65 | Excellent |
| 25 | -63 | Excellent |
| 26 | -61 | Excellent |
| 27 | -59 | Excellent |
| 28 | -57 | Excellent |
| 29 | -55 | Excellent |
| 30 | -53 | Excellent |

从这个表可以得出以下几个结论：

1. CSQ的变化粒度是2dBm。
2. 信号质量的划分，0~9 （-113~-95dBm）信号差；10~14（-93~-85dBm）信号一般；15~19 （-83~-75dBm）信号好；20~30（-73~-53dBm）信号很好。