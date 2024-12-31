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

Nand Flash 里面有一个缓存，读写 page 都要先操作 cache 内存，然后再从 cache 写入到flash里面或者从 cache 读出数据。 

- 内存寻址

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241231/image.3rbax1n6vp.webp)

总共4096个block, 每个block 64 个page, 每个page 4096+256字节。
内存地址分行地址RA和列地址CA，CA 13 bit，用来寻址page里面的每个字节的位置，RA 前6 bit 表示block中地几个page的地址，RA高12 bit表示第几个block的地址。

单个page里面的内存布局，前面 4096 字节的数据存储区域，后面 256字节的 spare area。
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20241231/image.2oblm6c5n5.webp)


## 指令集

| 命令    | 命令传输方式 | 地址长度bytes | 地址传输方式 | dummy 字节 | 数据传输方式 | 备注                                                         |
| ------- | ------------ | ------------- | ------------ | ---------- | ------------ | ------------------------------------------------------------ |
| 04H     | 单线         | 0             |              | 0          |              | Write Disable                                                |
| 06H     | 单线         | 0             |              | 0          |              | Write Enable                                                 |
| D8H     | 单线         | 3             | 单线         | 0          |              | Block Erase (Block size)<br />24-bit address 由 6 dummy bits and 18  block address bits 组成<br />RA地址中的page地址会被自动忽略 |
| 02H     | 单线         | 2             | 单线         | 0          | 单线         | Program Load                                                 |
| 32H     | 单线         | 2             | 单线         | 0          | 四线         | Program Load x4 IO<br />需要设置QE bit                       |
| 10H     | 单线         | 3             | 单线         | 0          | 单线         | Program Execute                                              |
| 84H     | 单线         | 2             | 单线         |            | 单线         | Program Load Random Data                                     |
| C4H/34H | 单线         | 2             | 单线         | 0          | 四线         | Program Load Random Data x4 IO<br />需要设置QE bit           |
| 72H     | 单线         | 2             | 四线         | 0          | 四线         | Program Load Random Data Quad IO<br />需要设置QE bit         |
| 13H     | 单线         | 3             | 单线         | 0          |              | Page Read (to Cache)<br />24-bit address consists of 6 dummy bits and 18 page address bits |
| 03H/0BH | 单线         | 2             | 单线         | 0          | 单线         | Read from Cache x1 IO<br />16 bits of column address which consists of 3 wrap bits and 13 column address bits |
| 3BH     | 单线         | 2             | 单线         | 1          | 双线         | Read from Cache x2 IO                                        |
| 6BH     | 单线         | 2             | 单线         | 1          | 四线         | Read from Cache x4 IO                                        |
| BBH     | 单线         | 2             | 双线         | 1          | 双线         | Read from Cache Dual IO                                      |
| EBH     | 单线         | 2             | 四线         | 1          | 四线         | Read from Cache Quad IO<br />需要设置QE bit                  |
| 9FH     | 单线         | 1             | 单线         | 0          | 单线         | Read ID<br />address 00H： manufacturer ID<br />address 01H： device ID<br />datasheet 这两个ID是分开的地址，时序图是分开读取的，但实际驱动可以连续读2字节。 |
| FFH     | 单线         |               |              |            |              | Reset                                                        |
| 0FH     | 单线         | 1             | 单线         | 0          | 单线         | Get Feature                                                  |
| 1FH     | 单线         | 1             | 单线         | 0          | 单线         | Set Feature                                                  |

## 功能寄存器


## 操作时序



## 驱动



## 参考资料

- [SPI NAND Flash Driver](https://github.com/espressif/idf-extra-components/tree/57c8917e0204f0058863b3bc67517183cd8ae71e/spi_nand_flash)
- [AllianceMemory_SPI_NAND_Flash_July2020_Rev1_0-1893515.pdf](https://www.mouser.com/datasheet/2/12/AllianceMemory_SPI_NAND_Flash_July2020_Rev1_0-1893515.pdf)
- [Dhara: NAND flash translation layer for small MCUs](https://github.com/dlbeer/dhara)
