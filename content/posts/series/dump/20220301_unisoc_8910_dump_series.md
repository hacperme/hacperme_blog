---
title: "展锐 8910 平台 dump 抓取和分析专题"
date: 2022-03-01T00:55:59+08:00
lastmod: 2023-03-27T00:55:59+08:00
author: ["hacper"]
tags:
    - dump
    - trace32
    - 展锐
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "展锐 8910 平台 dump 分析培训分享内容。" # 文章简单描述，会展示在主页
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
# cover:
#     image: ""
#     caption: ""
#     alt: ""
#     relative: false

---

## 1. dump 抓取方法

- 抓取dump的前提

  抓取dump之前需要关闭模块的硬件看门狗，可以发at指令或者调用open sdk api

  AT 指令：

  ```
  at+qdbgcfg="dumpcfg",0,1
  ```

  API:

  ```c
  ql_dev_cfg_wdt(1);  // 打开硬件看门狗 
  ql_dev_cfg_wdt(0);  // 关闭硬件看门狗
  ```

- 使用coolwatcher工具抓取dump的步骤

  在coolwatcher 工具里面选择tools->blue screen dump。

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.6kl1y8cbopo0.png)

  elf 文件选择编译出来的 8915DM_cat1_open_core.elf，然后再选择dump的保存路径，点击start开始传输dump文件。

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.7ba9cp8hy100.png)

- Trace32 加载dump文件步骤

  解压压缩包T32.7z，把dump文件复制到8910dump目录下，同时需要复制编译固件的elf文件，同时放到8910dump目录下。

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.1beoxyunshvk.png)

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.68n5rt8flvc0.png)

  双击 T32_8910_Quectel_ap.bat 脚本进行解析

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.2gg1onaptj6s.png)

  

  

- 抓取dump失败的常见问题

  1. 设备的固件与解析的elf文件不匹配
  2. 使用串口抓取dump，需要串口支持921600波特率，否则只能用USB方式抓取。
  3. dump 无法导出，尝试使用GDB launch 工具查看调用栈。
  4. 调用栈函数名字没有解析出来，可能是没有加载客户app的elf文件，重新加载app.elf或者根据函数地址查找map文件。

## 2. TRACE32 工具的使用

- 查看调用栈

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.6197iyj56dw0.png)

- 查看寄存器

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.5333jcu67bg0.png)

  R15--PC,

  R14--LR,

  R13--SP

  

- dump 内存

  输入16进制内存地址

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.1b610cfkslcw.png)

  

- 线程列表

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.43dqywq15fw0.png)

  查看tcb

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.7bdb8l9q8vo0.png)

- 查看变量

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.5jmpgb7kr3k0.png)

  使用命令 v.v 变量类型 变量地址

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.5xhx3po5k6o0.png)

- 通过设置断点，根据函数地址查找函数名字

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.6j2i6nh6ioo0.png)

  

- 查看类型，数据结构定义

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.4h67d8yjesq0.png)
  
- 查看代码

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3c77p6zotde0.png)

## 3. 典型问题分析方法

### 3.1 访问非法内存死机

怎么判断是否是访问非法内存呢？

通过查看死机时的汇编指令(通常是内存寻址指令)和寄存器来判定。

示例如下图：

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.30mm21bknm60.png)

 ldrb    r2,[r1,#0x5]  为死机的位置，ldrb 是ARM汇编中的一条内存寻址指令，作用是将从存储器中将一个``8``位的字节数据传送到目的寄存器中，同时将寄存器的高``24``位清零。



ldrb    r2,[r1,#0x5]  的意思是 将存储器地址为R1＋0x5的字节数据读入寄存器R2，并将R2的高24位清零。也就是该指令访问了R1寄存器所指向的内存地址，而查看R1寄存器的值发现R1为0，也就是访问了空指针（0地址）。



### 3.2 栈溢出分析及栈溢出检测机制



栈溢出问题可以分为两种类型，一是栈空间不够，在压栈的时候使用的空间超出了任务的栈空间大小；另一种是局部变量溢出，导致栈空间保存的数据被破坏。

- 栈溢出检测机制

  展锐的8910使用的是FreeRTOS， 栈的增长方向是从高地址向地址增长。其栈溢出检测的原理是，在任务栈初始化的时候，填充固定数据0xA5，kernel 会检查任务栈的最后16个字节的数据是否被篡改，如果发现这16个字节不是0xA5，则会触发ApplicationStackOverflowHook的调用，然后调用osiPanic() 主动死机。

- 栈溢出分析示例

  栈空间大小分配不够导致栈溢出的问题的判断方法：

  这种类型的栈溢出判断比较简单，直接看Trace32的调用栈里面是否有vApplicationStackOverflowHook的调用；或者查看死机task的TCB，查看其栈的起始地址对应的内存数据，看填充的A5A5数据是不是被篡改了，如果不是A5A5，则可以断定是栈溢出了。

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.705487lxxtg0.png)

  

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.1yppd8g70s9s.png)

  局部变量溢出导致dump的判断方法：

  这类问题比较难定位到具体是哪个变量溢出，但表现的现象很有特点：看任务栈的空间没有超，且每次死机的位置不固定，或者PC寄存器指向的地址不是一个正常的函数地址。遇到这类情况可以往局部变量溢出导致栈被破坏这个方向排查问题。

### 3.3 内存写越界dump分析方法



判断内存越界的方法，如果有导出heap report文件，可以先看下heap report里面有没有 block tail pattern error 的地方。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.19zvvjkrafuo.png)

如果没有heap report文件，则可以从dump解析里面去判断，具体方法如下：

先看调用栈里面有没有prvFree.part.5的调用，如果有则有可能是检测到内存被破坏了。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.x7rh1xs5d5s.png)

对于指定的内存地址，看有没有越界，还可以通过dump内存数据来判断，例如查看0x80BAEC90这个地址有没有被破坏：

在命令窗口执行如下命令，对内存地址转换为struct osiBlockHeader *变量来查看：

```
v.v (struct osiBlockHeader *)(0x80BAEC90-8)
```

注意对地址0x80BAEC90 向前偏移8个字节

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3whwy6coo960.png)

我们可以得到几个信息：

caller = 806162601

size = 4,

对caller = 806162601（十进制）乘以2，可以得到申请这段内存的函数地址，806162601*2=1612325202 （0x601A2152），通过设置断点的方式可以找到地址0x601A2152对应的函数是mlConvertStr。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.52wsocwbxq40.png)

size = 4（十进制）乘以8，可以得到这段内存的大小，4*8=32 （0x20）

然后dump这段内存，查看这段内存的tail位置是否为fd，如果不是则表示越界了

查看方法，输入指令:

```
d.dump (0x80BAEC90-0x08+0x20-0x01)
```

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3vk1kk62we80.png)

看箭头所指的地方是否为fd。

tail的位置的计算方法，内存地址-0x08+内存大小-0x01

上述dump内存的方法和导出的heap report是可以对应的上的：

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3xcjcommx8o0.png)



​	取一个正常的内存地址做对比，以dump中的地址0x80BAEC28为例：

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.1xhhsmxcpeow.png)



![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3yp6fparxdm0.png)

caller = 806521247,  806521247*2 = 1613042494（0x6025133E）

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.4ve7tv6clp80.png)

size = 13,  13*8=104 (0x68)

查看tail

```
d.dump (0x80BAEC28-0x08+0x68-0x01)
```

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3pptmlkjmgo0.png)

内存的结构：

```
 8字节header(包含caller，size等信息)+数据区(malloc返回的地址)+8字节对齐的冗余数据+1字节的尾部（其中最后一个字节为tail: FD）
```

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.4spx3x78qqe0.png)
