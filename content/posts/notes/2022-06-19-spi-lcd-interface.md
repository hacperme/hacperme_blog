---
id: 1962
title: '常见的 SPI LCD 接口分类'
date: '2022-06-19T11:53:33+08:00'
author: hacper
excerpt: '介绍 spi 串口 LCD 屏的接口分类和差异。'
layout: post
guid: 'https://hacperme.com/?p=1962'
permalink: /2022/06/19/spi-lcd-interface/
views:
    - '1537'
categories:
    - 笔记
tags:
    - lcd
    - spi
---

在评估一款芯片是否支持某款屏幕时，首先要看屏幕的通信接口是否支持，认识和了解屏幕接口的类型，是能否正确评估屏幕选型和驱动调试的前提之一。下面介绍 spi 串口 LCD 屏的接口分类和差异。

通常带 spi LCM 外设的芯片，其屏幕的 spi 接口并不是标准的 4 线 spi（四根信号线：CS, MOSI, MISO, CLK），而是专用的 spi 接口。

spi 串口 LCD 屏的接口分类和差异整理之后，如下表所示：

| <th>信号线</th> | <th>4 线 8 bit Ⅰ 型 1 data line</th> | <th>4 线 8 bit Ⅱ 型 1 data line</th> | <th>3 线 9 bit Ⅰ型 1 data line</th> | <th>3 线 9 bit Ⅱ型 1 data line</th> | <th>3 线 9 bit Ⅰ型 2 data line</th> | <th>3 线 9 bit Ⅱ型 2 data line</th> |
|--------------------|------------------------------------------|------------------------------------------|-----------------------------------------|-----------------------------------------|-----------------------------------------|-----------------------------------------|
| CS | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| CLK | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| RS(D/CX) | ✔(数据/命令选择) | ✔(数据/命令选择) | ✖ (RS(D/CX)被嵌入数据流中, 没有引脚) | ✖ (RS(D/CX)被嵌入数据流中, 没有引脚) | ✖ (RS(D/CX)被嵌入数据流中, 没有引脚) | ✖ (RS(D/CX)被嵌入数据流中, 没有引脚) |
| DIN | ✖ | ✔ | ✖ | ✔ | ✖ | ✔ |
| DOUT | ✔（DIO, 数据输入/输出） | ✔ | ✔（DIO, 数据输入/输出） | ✔ | ✔（D0） | ✔ |
| DOUT2 | ✖ | ✖ | ✖ | ✖ | ✔（RS(D/CX) 复用为 D1）第二 bit 输出 | ✔（RS(D/CX) 复用为 D1）第二 bit 输出 |

## 总结

1. 4 线 spi 和 3 线 spi 接口的区别，4 线 spi 有 D/CX 引脚用于控制命令还是数据选择，而 3 线 spi 没有 D/CX 引脚，在数据中使用 1bit表示命令还是数据。

  ![](https://raw.githubusercontent.com/hacperme/picx_hosting/master/20210507/xxx.1fhb0avtpucg.png)
2. 1 data line 和 2 data line 的区别，1 data line 数据位宽只有 1bit, 2 data line 数据位宽有 2 bit, RS(D/CX) 复用为 D1，作为第 2 bit data。

  ![](https://raw.githubusercontent.com/hacperme/picx_hosting/master/20210507/xxx.1fhb0avtpucg.png)

3. Ⅰ 型 和 Ⅱ 型屏的区别，Ⅱ 型屏有独立的 DIN、DOUT 两个引脚，而Ⅰ 型屏只有一个 data （DIO）引脚，既做输入又做输出。