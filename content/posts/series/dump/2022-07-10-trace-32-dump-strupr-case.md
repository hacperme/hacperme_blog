---
id: 1964
title: 'trace 32 dump 分析案例：strupr 修改只读内存引发的血案'
date: '2022-07-10T00:41:37+08:00'
author: hacper
excerpt: '介绍一个使用 trace 32 分析 dump 的案例，通过这个案例熟悉 trace 32 工具的使用方法，并了解常见的 dump 问题排查思路。'
layout: post
guid: 'https://hacperme.com/?p=1964'
permalink: /2022/07/10/trace-32-dump-strupr-case/
views:
    - '510'
categories:
    - 笔记
tags:
    - dump
    - trace32
---

## 前言

介绍一个使用 trace 32 分析 dump 的案例，通过这个案例熟悉 trace 32 工具的使用方法，并了解常见的 dump 问题排查思路。

死机的设备是展锐 8910 平台，操作系统是 freertos, c 语言开发环境。

## 问题分析过程

打开 trace 32，加载设备死机时 dump 文件， 首先开任务调用栈，看死机时的停留位置。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image.bty9ryslgd.png)

死机正在调用的一个函数为strupr，对应的汇编代码为 strb r3,\[r1\]，这种情况，按照以往经验可以猜测大概率是内存读写问题。为了避免误判，还是需要做一下常规异常排查。

1. 查看任务列表，按照任务状态排序，查看ready状态的任务是否有看门狗喂狗任务，排除高优先级任务占用cpu，喂狗任务无法及时得到调度而死机的情况，从下面的截图看呢，喂狗任务正常挂起了，所以不是这个原因

  ![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image.7i6s6qfuzaw0.png)
2. 查看任务栈，粗略看下有没有栈溢出的情况，排除是否是栈溢出导致内存访问问题。

  ![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image.6j9ebcbbjn00.png)

  可以看到，download 任务的任务栈还有 0x2c38 字节未使用，任务栈是够用的，可以排除栈溢出问题。

排除这两种常见情况之后，再分析死机异常的具体原因。

查看死机时汇编代码 strb r3,\[r1\]，这条指令有访问r1寄存器所指向的内存数据，查看r1寄存器的值，为0x601FFC8C，根据平台经验，0x601FFC8C 这个地址要么是一个属于代码段的地址，要么是一个不存在的非法内存地址，因为栈或者堆上的内存地址是长成0x80xxxxxx这样的。

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.5naug3dlcpo0.png?raw=true)

可以dump一下0x601FFC8C这个地址的数据，看看里面有什么内容，可以看下面截图，0x601FFC8C 地址存放的刚好是一个字符串 “battery”，0x601FFC8C正好是这个字符串的起始地址，由于0x601FFC8C是属于代码段的地址，只能读取不能修改，所以可以断定0x601FFC8C是一个字符串常量。死机原因也可以进一步推测，是不是修改了这个只读变量的内存数据而导致死机呢？

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.4da396he42y0.png?raw=true)

再来看 strupr 这个函数的功能，strupr 是 c 标准库中的一个函数，功能是输入一个字符串数据，然后将其转换为大写。

问题原因似乎逐渐清晰了，汇编指令 strb 的功能正好是将寄存器的一个字节保存到某个内存地址，strb r3,\[r1\] 也就是将 r3的值保存到r1所指向的地址0x601FFC8C，r3的值是0x42, 对应ascii 字符 “B”, 目的是将battery中的“b”改成大写的"B"。

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.48toxjsm38g0.png?raw=true)

0x601FFC8C 地址里的数据是只读的，修改0x601FFC8C里面的数据导致了死机，死机的根本原因也就找到了。

最后再来看看这段触发死机的代码是怎么写的，cmd\_get\_config\_info 函数里面调用了strupr函数死机，需要查看cmd\_get\_config\_info 的实现

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.2ikctxfarwo0.png?raw=true)

cmd\_get\_config\_info 中使用 strupr 修改数组 names 里面的值

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.1y35mq911fog.png?raw=true)

然后再看 names 里面的数据怎么来的

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.1e8zw48ib9r4.png?raw=true)

原来啊，names 里面的数据，是来自 static const char \* 定义的一组字符串数据。在 trace 32 里面，也可以通过对names这个变量指向的地址0x80FA0B94设置断点的方式，查看变量名或者函数名，如下图所示，0x80FA0B94正好是全局变量 \_\_HARDWARE\_CONFIG\_MODULES 的起始地址。

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/image.4rbb4r2qsf40.png?raw=true)

## 一点感想

c 语言是不安全的编程语言，开发调试过程中出现内存问题不可避免，无论如何小心谨慎，老司机也会有翻车的时候，c 语言这样的语言特性，对开发维护和支持来说，是一个不小的包袱。

## 扫码关注

![](https://git.poker/hacperme/picx_hosting/blob/master/20210507/qrcode_for_gh_b1444a13ac67_258.79qtoo80p9s0.jpg?raw=true)