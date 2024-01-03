---
title: "cmake 构建环境下自定义 section 失效的问题"
date: 2023-12-17T17:04:49+08:00
lastmod: 2023-12-17T17:04:49+08:00
author: ["hacper"]
tags:
    - cmake
    - section
    - RW612
    - __attribute__
    - GCC
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "cmake 构建系统下通过 __attribute__((section(\".xx\"), used)) 指定代码存储位置，保留代码被优化删除的问题。" # 文章简单描述，会展示在主页
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


由于项目需要，前几天在调整 NXP RW612 MCU wifi SDK 的构建系统，由原来的 cmake+mingw+make+armgcc 调整为 cmake+kcongfig+ninja+mingw+armcc。构建系统调整完成之后，发现烧录固件设备无法启动，而未调整之前编译出来的固件是正常的。

## 查找原因

对比二者编译生成的固件镜像文件，旧构建系统生成的固件比新构建系统生成的固件文件大小大一点。首先怀疑的是否是一些宏定义没启用，部分代码被屏蔽了而未编译进去，经过一些测试和检查之后发现不是这个问题。接下来从map文件着手，对比发现map文件有差异。

旧构建系统的map信息：
```bat
.flash_config   0x08000000     0x1000
                0x08000000                . = ALIGN (0x4)
 FILL mask 0xff
                0x00000400                . = 0x400
 *fill*         0x08000000      0x400 ff
                0x08000400                __FLASH_BASE = .
 *(.flash_conf)
 .flash_conf    0x08000400      0xc00 out/libs/libmcux_sdk_module_lib.a(flash_config.c.obj)
                0x08000400                flexspi_config
                0x00001000                . = 0x1000

.interrupts     0x00001000      0x280 load address 0x08001000
                0x00001000                . = ALIGN (0x4)
                0x00001000                __VECTOR_TABLE = .
                0x00001000                __Vectors = .
```

新构建系统的map信息：

```bat
.flash_config   0x08000000     0x1000
                0x08000000                . = ALIGN (0x4)
 FILL mask 0xff
                0x00000400                . = 0x400
 *fill*         0x08000000      0x400 ff
                0x08000400                __FLASH_BASE = .
 *(.flash_conf)
                0x00001000                . = 0x1000
 *fill*         0x08000400      0xc00 ff

.interrupts     0x00001000      0x280 load address 0x08001000
                0x00001000                . = ALIGN (0x4)
                0x00001000                __VECTOR_TABLE = .
                0x00001000                __Vectors = .
```

旧的构建系统有将 flash_config.c.obj 的代码 flexspi_config 存储在 .flash_config 这个 section，而新的固件却没有，整个map文件都查不到 flexspi_config  的信息。根据 flash_config.c 的代码在这篇文章[痞子衡嵌入式：深扒i.MXRTxxx系列ROM中集成的串行NOR Flash启动SW Reset功能及其应用场合](https://www.cnblogs.com/henjay724/p/15085155.html)中找到 flexspi_config 的作用：flexspi_config 是固件镜像的启动头 FDCB，bootrom 用来给 FlexSPI 外设初始化 flash 用的。对比两个镜像文件也能看到差异，一个有 FDCB，一个没有。
![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image.6so0kh3g44w0.webp)

那么 FDCB 是怎么生成的呢？

查看 RW612 的 ld 文件：
```ld

/* Entry Point */
ENTRY(Reset_Handler)

HEAP_SIZE  = DEFINED(__heap_size__)  ? __heap_size__  : 0x400;
STACK_SIZE = DEFINED(__stack_size__) ? __stack_size__ : 0x400;

/* Specify the memory areas */
MEMORY
{
  m_flash               (RX)  : ORIGIN = 0x08000000, LENGTH = 0x00200000
  m_interrupts          (RX)  : ORIGIN = 0x00001000, LENGTH = 0x00000280
  m_text                (RX)  : ORIGIN = 0x00001280, LENGTH = 0x000B9080
}
/* 0x2012_3000 - 0x2012_FFFF is reserved for BootROM usage */

/* Define output sections */
SECTIONS
{
  .flash_config :
  {
    . = ALIGN(4);
    FILL(0xFF)
    . = 0x400;
    __FLASH_BASE = .;
    KEEP(* (.flash_conf))     /* flash config section */
    . = 0x1000;
  } > m_flash

  /* The startup code goes first into internal ram */

  /*....*/
}
```
在flash的起始地址定义了一个 4K大小的 .flash_config section，前面填充 1K 0xFF, 后面开始存储 flash_conf 的代码。
在 boards\rdrw612bga\flash_config\flash_config.c 也指定了将代码放到 flash_conf section： __attribute__((section(".flash_conf"), used))

```c
#if defined(BOOT_HEADER_ENABLE) && (BOOT_HEADER_ENABLE == 1)

#if defined(__ARMCC_VERSION) || defined(__GNUC__)

__attribute__((section(".flash_conf"), used))

#elif defined(__ICCARM__)

#pragma location = ".flash_conf"

#endif

const fc_flexspi_nor_config_t flexspi_config = {
    .memConfig =
        {
            .tag                 = FC_BLOCK_TAG,
            .version             = FC_BLOCK_VERSION,
            .readSampleClkSrc    = 1,
            .csHoldTime          = 3,
            .lookupTable =
                {
                    /* Read */
                    [0] = FC_FLEXSPI_LUT_SEQ(FC_CMD_SDR, FC_FLEXSPI_1PAD, 0xEC, FC_RADDR_SDR, FC_FLEXSPI_4PAD, 0x20),
                    [1] = FC_FLEXSPI_LUT_SEQ(FC_DUMMY_SDR, FC_FLEXSPI_4PAD, 0x0A, FC_READ_SDR, FC_FLEXSPI_4PAD, 0x04),
                    /* chip erase */
                    [4 * 11 + 0] =
                        FC_FLEXSPI_LUT_SEQ(FC_CMD_SDR, FC_FLEXSPI_1PAD, 0x60, FC_STOP_EXE, FC_FLEXSPI_1PAD, 0x00),
                },
        },
    .pageSize           = 0x100,
    .sectorSize         = 0x1000,
    .ipcmdSerialClkFreq = 0,
    .blockSize          = 0x8000,
    .fcb_fill[0]        = 0xFFFFFFFF,
};

#endif /* BOOT_HEADER_ENABLE */

```

那么为什么 FDCB 在新的构建系统下没有生成呢？

我发现二者的差异在于旧构建系统是将flash_config.c放到了构建可执行文件目标的源码里面 add_executable。而新的构建系统是将 flash_config.c 放到构建静态库的源码里面 add_library，然后在构建可执行目标文件的时候通过添加依赖的lib target_link_libraries 加入，在最终链接的时候由于 flash_config.c 的代码未使用而被删除了，即使配置了 __attribute__((section(".flash_conf"), used)) 也会删除，这个应该是 cmake 的问题。

解决办法有两个：

1. 将 flash_config.c 放到构建可执行目标的源码里面。
2. 找个地方调用一下 flash_config.c 里的变量或者函数，避免未使用而被优化。


使用 cmake 构建系统时需要留意这个坑，如果需要自定义 section 指定变量或者函数的存储位置，要留意代码有没有被使用和是否是在构建可执行的目标的源码中，不然即便设置了 __attribute__((section(".xx"), used)) 也还是会被优化删除。


## 资料

- [痞子衡嵌入式：深扒i.MXRTxxx系列ROM中集成的串行NOR Flash启动SW Reset功能及其应用场合](https://www.cnblogs.com/henjay724/p/15085155.html)