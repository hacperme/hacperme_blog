---
title: "SPI NAND Flash AS5F38G04SND-08LIN 笔记"
date: 2024-12-30T00:49:37+08:00
lastmod: 2024-12-30T00:49:37+08:00
author: ["hacper"]
tags:
    - SPI
    - NAND
    - FLASH
    - AS5F38G04SND-08LIN
categories:
    - 笔记
description: "SPI NAND Flash AS5F38G04SND-08LIN 驱动相关笔记" # 文章描述，与搜索优化相关
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

## AS5F38G04SND-08LIN flash 特点

- Single-Level Cell (SLC) NAND Flash
 SLC 和 MLC 的区别：SLC (Single-Level Cell)：每个存储单元只存储 1 位数据（0 或 1）。这使得 SLC 比 MLC 更可靠、更耐用。MLC (Multi-Level Cell)：每个存储单元可以存储 2 位数据（通常是 4 个电压级别，代表 00、01、10、11）。因此，MLC 的存储密度更高，但性能和耐用性略逊于 SLC。

- Clock Frequency
 Up to 120MHz (for VCC 3.3V)

- 支持 Standard, Dual and Quad SPI

- 8bit ECC for each 512bytes + 32bytes
- 容量
 Density：8Gbits
 ECC：8bit
 Page Size：4096(data storage region)+256(spare area) Bytes，data storage region 用来存储数据，spare area 用来做内存管理和数据纠错。
 Block: 64 Pages
 Device:4096 Blocks

- 功能框图
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241230/image.70aeso4j53.webp)

## 指令集

## 驱动
