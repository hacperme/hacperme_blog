---
title: "无锁数据结构 lock-free 笔记"
date: 2023-05-13T08:48:34+08:00
lastmod: 2023-05-13T08:48:34+08:00
author: ["hacper"]
tags:
    - lock-free
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: true # 是否为草稿
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

## 介绍

无锁数据结构是一种线程和中断安全的数据结构，无需使用互斥机制，用于进程、线程间通信。

那么，解决多任务之间数据竞争的方式、使用互斥和无锁数据结构？

lockfree 中的所有结构都是基于数组的、有界的、无锁的，适用于单消费者单生产者场景。

在单生产者单消费者场景下都是线程和多核安全的。

有哪些数据结构？

- queue

  适用于单元素操作

- ringbuffer

  一种更通用的数据结构，能够一次处理多个元素

  每当需要一次添加或删除多个元素时
  对于需要排队但每次发送n个字节的可变大小数据包的字节流
  当数据操作可能失败或仅使用部分数据时

- bipbuffer(二分缓冲区)

  环形缓冲区的变体，能够始终在缓冲区中提供线性空间，使得缓冲区内处理成为可能

使用无锁数据结构的原因

- 性能

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-20230513094637948.20qv3ct73slc.webp)

## 参考资料

- https://github.com/DNedic/lockfree
- 《Lock-Free Programming (or, Juggling Razor Blades) - Herb Sutter - CppCon 2014》