---
title: "windows 10 设置网卡优先级"
date: 2023-05-14T00:12:17+08:00
lastmod: 2023-05-14T00:12:17+08:00
author: ["hacper"]
tags:
    - windows
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
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

工作中需要连接内网（以太网）和访客网络（WiFi），内网有一些限制，会有一些网站无法访问。同时接入内网和外网，有时候上网网络默认会走内网路由导致无法上网，所以不得不临时禁用内网，此时内网机器又会断连...，浪费时间在两个网络切来切去。

后面想了一下，是否可以设置上网时的网卡的优先级呢，优先使用WiFi的访客网络，其次才用内网。网上查了一下方法，还真可以。

## 设置网卡优先级方法

其实是通过修改路由的跃点数来修改网络的优先级的，跃点越小，优先级越高，按照下图，在控制面版将外网的跃点数修改到小于内网的跃点数，优先使用外网。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image.41b7n08kiyi0.webp)

内网访问，再添加固定路由：

```powershell
route add -p 192.168.92.208 mask 255.255.255.0 192.168.92.13 metric 3
```



## 参考

- [windows双网卡时设置网络优先级](https://lvbibir.cn/archives/791)