---
title: "TRACE32 中通过任务栈中保存的数据恢复 freertos 某个任务调用栈的方法"
date: 2024-07-31T02:00:18+08:00
lastmod: 2024-07-31T02:00:18+08:00
author: ["hacper"]
tags:
   - FC41D
   - freertos
   - bk7231m
   - Trace32
   - dump
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "介绍怎么通过tcb中栈的数据恢复任务调用栈。" # 文章简单描述，会展示在主页
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

T32 加载 dump 之后看不到任务调用栈，是因为寄存器的值异常，导致任务调用栈没解析出来。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240731/image.7p3ib6p4y8.webp)

可以通过 tcb 中栈数据恢复寄存器的值：
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240731/image.3k7wz2uwfy.webp)

打开对应的 task struct，dump pxTopOfStack(0x411420) 指向的内存数据
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240731/image.4jo0c92xsd.webp)

在pxTopOfStack(0x411420)的第8字节开始，对应寄存器 r1 ~ r15(pc)的值，分别将这些值设置到寄存器。
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240731/image.73tuowfe49.webp)

这样任务调用栈便恢复了，可以继续进行下一步的分析。
