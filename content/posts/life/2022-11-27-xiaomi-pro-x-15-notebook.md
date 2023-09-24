---
id: 1974
title: '小米笔记本 PRO X 15 熄屏后无法点亮屏幕问题处理'
date: '2022-11-27T16:31:08+08:00'
author: hacper
layout: post
guid: 'https://hacperme.com/?p=1974'
permalink: /2022/11/27/xiaomi-pro-x-15-notebook/
views:
    - '603'
categories:
    - 生活
tags:
    - Windows
---

小米笔记本，win10 系统，笔记本在前段时间使用的时候发现一个问题：系统休眠、睡眠或者熄屏之后屏幕无法点亮，只能强制重启，但重启之后熄屏还是会出现无法亮屏的情况，差点就送去维修了。

后面自己瞎折腾，怀疑是显卡驱动的问题，也算是运气好，居然修好了。在设备管理器查看显卡驱动，发现显卡驱动显示未启动 igfxn 设备。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/1.2ifq8fzpiuw0.png)

然后尝试删除设备，已经黑屏了，只能强制重启，重新开机之后可以看到 igfxn 设备启动正常，熄屏之后无法点亮屏幕的问题也消失了。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/2.2h5jrnqhska0.png)