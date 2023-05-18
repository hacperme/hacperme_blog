---
title: "DRAM \"Row Hammer\" 问题"
date: 2023-05-18T23:11:21+08:00
lastmod: 2023-05-18T23:11:21+08:00
author: ["hacper"]
tags:
    - pSRAM
    - DRAM
    - Row Hammer
    - DUMP
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "最近遇到一个在 RAM 中某个比特位数据翻转导致的死机问题，在排查解决过程中进而了解到 DRAM \"Row Hammer\" 现象。" # 文章简单描述，会展示在主页
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

## 背景

最近遇到一个内存被破坏的死机问题，一开始排查以为是应用程序踩内存导致 heap 被破坏，后面排查发现是内存中某一个比特位发生翻转，由 0 变成 1，或者由 1 变成 0，导致程序读取数据异常触发死机，最后靠提供 pSRAM 的刷新率来规避该问题。在解决这个问题的过程中了解到 DRAM 的 "Row Hammer" 现象，我也是第一次遇到，做一下知识总结。

## 什么是 "Row Hammer" 现象？

"Row Hammer" 翻译过来叫 “行锤”，是目前所有 DRAM 类型存储器存在的一个硬件问题。"Row Hammer" 现象是指对 DRAM 的某一行内存高频率地反复进行读写，会对其相邻行内存造成冲击。如果在一个刷新周期内对同一个内存单元读写超过了一定次数，那么可能导致相邻的内存单元的位状态改变，从而造成数据丢失和损坏，这并不是永久性的内存数据破坏，正常情况下，下一次刷新时间到了，可以恢复原来正常数据状态。

## "Row Hammer" 产生的原因

随着 DRAM 制造工艺的发展，DRAM 的存储密度不断增大，内存单元体积也不断缩小，其存储的电荷数量也随之减小，内存单元之间噪声容限降低，导致相互独立的 2 个内存单元之间的电荷相互影响。如果反复高频率地访问同一行内存，则来自过度激活行的电荷会泄漏到相邻行中，导致位反转现象，即0反转为1，1反转为0。

## 规避方法

一种简单有效的方法是提高 DRAM 的刷新率，刷新率越高，在刷新周期内的读写次数越少，从而减少出现 Row Hammer 的概率。提高刷新率的缺点是会干扰 DRAM 的访问，减慢系统速度，同时增加内存的功耗。

其他规避方法：
- ECC（error corrected code），缺点也是会降低系统访问速度，同时增加成本。
- PARA（probabilistic adjacent row activation），PARA技术的主要原理是每当一个行关闭时，内存控制器都会以一定的概率来刷新该行邻接行的值
- TRR（target row refresh），这种技术采用了一种特殊模块，从而可以跟踪记录内存中哪些行经常被激活，并且刷新这些激活行相邻行的值。


## 参考资料

- [What is DRAM “Row Hammer”?](https://thememoryguy.com/what-is-dram-row-hammer/)
- [Row Hammer漏洞攻击研究](http://www.infocomm-journal.com/cjnis/article/2018/2096-109x/2096-109x-4-1-00069.shtml)