---
title: "使用 trace32 分析 use after free dump 问题"
date: 2023-05-03T10:02:07+08:00
lastmod: 2023-05-03T10:02:07+08:00
author: ["hacper"]
tags:
    - trace32
    - use after free
    - dump
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "介绍 use after free 导致 dump 的两个案例。" # 文章简单描述，会展示在主页
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



## 介绍

use after free 是一种常见的内存安全问题，指的是在程序释放了某个内存块之后，仍然继续使用该内存块，导致程序崩溃或者出现其他安全问题。

use after free 问题其实包含以下情况：

- 内存块被释放后，指向该内存块的指针被设置为 NULL ，然后再次使用，导致程序访问非法地址而崩溃。
- 内存块被释放后，指向该内存块的指针没有被设置为 NULL ，在这块内存没有再次使用之前，如果没有代码修改这块内存，那么可能程序仍能正常运行。但如果这块内存在它下一次使用之前被其他代码修改了内容，那么程序容易出现奇怪的问题，比如 heap 被破坏。

## 案例分析

### 案例一



先看第一个案例，内存块被释放后访问空指针的问题。

查看任务调用栈和死机原因，quec_rtos_queue_wait 的入参 msgQRef 为 0，传入了一个空指针NULL，导致后面的数据处理访问了非法内存。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/screenshot-1.37th3dolcsc0.webp)

那么这个空指针是怎么来的呢？需要结合源码和任务调度信息来确认。

先看线程调度情况，dump 时正在执行的的任务是 tts_work 这个任务，上一个执行的任务是 task_app。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-2023-04-28-16-34-08-060.75p72rtzr800.webp)

再看 task_app 的任务调用栈，在后面看到这个任务调用 quec_tts_deinit 和调用了删除消息队列的函数

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-2023-04-28-16-35-33-855.4ngdkadkbiu0.webp)

tts_work 的任务栈中调用quec_rtos_queue_wait之后就出现 assert。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-2023-04-28-16-34-43-231.6mj9c27tilg0.webp)

从上面可以猜测，问题原因应该是在 task_app 任务中删除了消息队列，将消息队列的句柄设置为NULL，然后刚好内核进行了一次任务调度，切换到了 tts_work 任务运行，tts_work 里面没有判断消息队列句柄是否有效，一调用接收消息队列接口便访问了空指针。

再看 quec_tts_deinit 的代码逻辑，去初始化的时候，先释放了内存资源，后面才删除任务，没想到删除任务之前这个任务还会运行。可能接口设计者没有考虑到 quec_tts_deinit  这里面的代码执行过程中会发生任务切换吧，导致了这个 use after free 的问题。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-2023-04-28-16-38-11-549.3sv21pq9r5k0.webp)

### 案例二



再看第二个案例，内存块被释放后指向该内存块的指针未设置NULL，其他程序修改这段内存导致 dump 问题。

通过 trace32 查看任务调用栈和 dump 原因，也是访问 0 地址问题，quec_rtos_queue_release 发送消息的时候传入的地址不是一个正常的内存地址，看上去像内存被踩。然后查看队列列表，果然有一个消息队列的控制数据被破坏。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.bqq0ey6mxlc.webp)

内存被破坏问题，需要解析 heap 使用情况，发现确实是内存被踩，但不是消息队列句柄相关内存被破坏，而是quec_rtos_mutex_create 申请的内存写越界了。

```
0x7e0bbea0 allocated 64 bytes by InvalidTaskName, func=quec_rtos_mutex_create                     CORRUPTED block(tail guard @0x7e0bbeec)!!
```

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-2023-03-25-22-32-08-094.57ievnbpr4g0.webp)

再从 t32 看互斥锁列表（实际看信号量列表，在这个平台互斥锁也是通过信号量来实现的），果然有一个不正常的数据 7E0BBEC0。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.2ijo0zox1y20.webp)

知道 7E0BBEC0 这个地址，怎么确定是那段代码创建的这个互斥锁呢？

根据经验，在创建信号量、消息队列、任务等这些操作句柄的时候，通常会定义成全局变量，可以尝试在全局变量里面查找是否有变量的值是 7E0BBEC0，从而反过来定位是哪段代码有问题。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.43t740omc4e.webp)

通过 7E0BBEC0 可以找到 ql_sleep_flags_mutex 这个变量，以及其相关的代码，检查这段代码逻辑之后，发现没有明显的有内存写越界的地方，也就是说这段代码本身没有问题，而是其他某个地方把这段内存修改了。

其实在查找 7E0BBEC0 这个地址是哪个变量的值时，发现了新的线索，除了 ql_sleep_flags_mutex 这个变量的值是 7E0BBEC0 之外，还有两个变量 f_pVEParamsTable、f_pGainParamsTable 的值也是 7E0BBEC0，三个指针变量指向了同一个地址，这就是问题所在。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.6ylvnqafnv80.webp)



![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.3vgzp18h9uy0.webp)

为什么三个变量会指向同一个地址呢？

想到的一个可能就是 use after free 问题。看了相关代码实现，发现是在操作 f_pVEParamsTable 变量时，malloc 申请了一块内存地址为 7E0BBEC0 的空间，把地址赋值给了变量 f_pVEParamsTable，做业务的时候也会把 f_pVEParamsTable 的值赋值给 f_pGainParamsTable  变量，做完业务之后，通过 free 释放了这块内存，但由于代码逻辑的缺陷，f_pVEParamsTable 和 f_pGainParamsTable  还保留着已释放内存 7E0BBEC0 的这个值。在创建 ql_sleep_flags_mutex 相关的互斥锁时，申请内存返回了 7E0BBEC0 这个地址给 ql_sleep_flags_mutex 变量，当下一次 f_pVEParamsTable  相关代码做业务时，向原来的 7E0BBEC0  这块内存写数据，但这块内存现在的所有者是 ql_sleep_flags_mutex，内存起始地址相同但分配的内存大小不同，一写数据就写越界了，最终表现的现象是刚开始的消息队列、互斥锁内存被踩。

## 总结

use after free 是一个常见的内存问题，由于 c 的语言特性，该问题不可避免。通过两个 use after free dump 案例的分享，一是加深对 use after free 问题的认识，二是也分享了一些排查问题思路和方法，通过任务调度顺序和任务运行的历史痕迹定位问题、通过全局变量的值定位问题代码，希望对你解决问题有所帮助。