---
title: "读取寄存器 data abort dump 案例"
date: 2021-10-31T00:25:00+08:00
lastmod: 2023-03-27T00:25:00+08:00
author: ["hacper"]
tags:
    - dump
    - trace32
    - EC100Y
    - ASR
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "读写寄存器 DataAbort dump 案例。" # 文章简单描述，会展示在主页
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

dump原因 DataAbort，**Error address (may not be relevant): D4280829**

```

Software version from AXF file: SDK_1.011.022
Software version from Dump files: SDK_1.011.022
------------------------------------------------------------------
Error description: DataAbort[AT 801A9D34],Unknown
Error address (may not be relevant): D4280829
Error thread: uiNguxTh, stack range (0x7E6AA140..0x7E6AE13B)
RTC time: 00.00.0000-00:00:00 (dd.mm.yy-hh:mm:ss)
Gasket registers: PESR=00000000, XESR=00000000, PEAR=00000000, FEAR=00000000, SEAR=00000000, GEAR=00
FAULT_STATUS=0x00000001
FAULT_ADDRESS=0xD4280829
------------------------------------------------------------------
```

任务调用栈信息，看调用栈，最后是在执行sdhci_dumpregs函数，data abort 的地址 0xD4280829。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.sn16avefy5c.png)

SDHC 寄存器基地址如下：

| SD  Host Controller Registers                             |
| --------------------------------------------------------- |
| The base addresses of the  Host Controller registers are: |
| SD1: 0xD4280000                                           |
| SD2: 0xD4280800                                           |
| SD3: 0xD4281000                                           |
| SD4: reserved                                             |

0xD4280829 是 0xD4280800 + 0x29，也就是操作了 SDHCI_POWER_CONTROL 寄存器。

```c
#define SDHCI_POWER_CONTROL			                    0x29

static void sdhci_dumpregs(UINT32 base)
{
	SDIOLOG_TRACE(": ============== REGISTER DUMP ==============\r\n");
	SDIOLOG_TRACE(": DMA addr: 0x%08x | Version:  0x%08x\r\n",
		sdio_readl(base, SD_SYSADDR_LOW_offset),
		sdio_readw(base, SDHCI_HOST_VERSION));
	SDIOLOG_TRACE(": Blk size: 0x%08x | Blk cnt:  0x%08x\r\n",
		sdio_readw(base, SD_BLOCK_SIZE_offset),
		sdio_readw(base, SD_BLOCK_COUNT_offset));
	SDIOLOG_TRACE(": Argument: 0x%08x | Trn mode: 0x%08x\r\n",
		sdio_readl(base, SD_ARG_LOW_offset),
		sdio_readw(base, SD_TRANSFER_MODE_offset));
	SDIOLOG_TRACE(": Present:  0x%08x | Host ctl: 0x%08x\r\n",
		sdio_readl(base, SD_PRESENT_STAT_0_offset),
		sdio_readw(base, SD_HOST_CTRL_offset));
	SDIOLOG_TRACE(": Power:    0x%08x | Blk gap:  0x%08x\r\n",
		sdio_readw(base, SDHCI_POWER_CONTROL), /*导致dump的地方*/
		sdio_readw(base, SD_BGAP_CTRL_offset));
	SDIOLOG_TRACE(": Wake-up:  0x%08x | Clock:    0x%08x\r\n",
		sdio_readb(base, SDHCI_WAKE_UP_CONTROL),
		sdio_readw(base, SD_CLOCK_CTRL_offset));
	SDIOLOG_TRACE(": Timeout:  0x%08x | Int stat: 0x%08x\r\n",
		sdio_readw(base, SD_SW_RESET_CTRL_offset),
		sdio_readl(base, SD_NORM_INTR_STS_offset));
	SDIOLOG_TRACE(": Int enab: 0x%08x | Sig enab: 0x%08x\r\n",
		sdio_readl(base, SD_NORM_INTR_STS_EBLE_offset),
		sdio_readl(base, SD_NORM_INTR_STS_INTR_EBLE_offset));
	SDIOLOG_TRACE(": AC12 err: 0x%08x | Slot int: 0x%08x\r\n",
		sdio_readw(base, SDHCI_ACMD12_ERR),
		sdio_readw(base, SDHCI_SLOT_INT_STATUS));
	SDIOLOG_TRACE(": Caps:     0x%08x | Max curr: 0x%08x\r\n",
		sdio_readl(base, SDHCI_CAPABILITIES),
		sdio_readl(base, SDHCI_MAX_CURRENT));
	SDIOLOG_TRACE(": Command:  0x%08x | RX_CFG_REG:0x%08x\r\n",
		sdio_readw(base, SD_CMD_offset),
		sdio_readl(base,SDHCI_RX_CFG_REG));
    SDIOLOG_TRACE("SDHCI_HOST_CTRL2: 0x%08x | PRESET_VALUE_FOR_SDR50:0x%08x",
        sdio_readw(base, SDHCI_HOST_CTRL2),
        sdio_readw(base, SDHCI_PRESET_VALUE_FOR_SDR50));
	SDIOLOG_TRACE(": ===========================================\r\n");
}

```

SDHCI_POWER_CONTROL 寄存器地址和描述如下：

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.75hikkip1jk0.png)

0xD4280800 + 0x29 对应 对应图中标黄区域。



sdio_readw(base, SDHCI_POWER_CONTROL) ，sdio_readw 读取两个字节的数据，对应地址范围：0xD4280829-0xD428082B。而其中0xD428082A-0xD428082B 已经是下一个寄存器的地址范围了。

```c
static UINT16 sdio_readw(UINT32 base, UINT32 reg)
{
    volatile UINT16* ptr16;
    UINT16 tmp;
    ptr16 = (volatile UINT16*)(base + reg);
    tmp = *ptr16;
    return tmp;
}
```

改成sdio_readb(base, SDHCI_POWER_CONTROL)  不会dump，sdio_readb 读取一个字节，也就是说，并不是0xD4280829这个地址不可用，而是访问数据的方式不正确导致dump。

```c
static UINT8 sdio_readb(UINT32 base, UINT32 reg)
{
    volatile UINT8* ptr8;
    UINT8 tmp;
    ptr8 = (volatile UINT8*)(base + reg);
    tmp = *ptr8;
    return tmp;
}
```
