---
title: "ASR CAT 1 平台申请不到内存dump案例"
date: 2022-03-01T00:48:52+08:00
lastmod: 2023-03-27T00:48:52+08:00
author: ["hacper"]
tags:
    - dump
    - ASR
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "内存泄漏导致内存申请失败。" # 文章简单描述，会展示在主页
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

内存耗尽导致的dump会有类似的错误输出：

```plaintext
Error description: (310,320,0),KIOSMEM,L:146
```

(310,320,0),KIOSMEM,L:146 表示申请 0x320(800)字节大小的内存失败。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/xxx.71ra56f17sc.png)

通过工具 Quec_Crane_Dump_Memory_Parse_Tool_v1.1.2 解析dump查看内存使用情况，打开output_osa_mem.txt，查找 `free block` 关键字，发现确实没有0x320（800）大小的空闲内存了。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/xxx.5mk1a5n2q58.png)

最后检查应用代码，定位发生内存泄漏的地方。

task QlUMaiRx 申请了365920字节，申请了10455次， 所以这里出问题的可能性最大，最后确定是串口接收回调 ql_uart_callback 内存泄漏。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/xxx.7c56drtmo4o0.png)
