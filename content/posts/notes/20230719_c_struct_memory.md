---
title: "C 的结构体成员在内存中的存储顺序以及对硬件的描述能力"
date: 2023-07-19T00:05:17+08:00
lastmod: 2023-07-19T00:05:17+08:00
author: ["hacper"]
tags:
    - c
    - 结构体
    - 位域
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "介绍 c 语言是如何通过指针、结构体、联合体、位域等这些语法来描述硬件的。" # 文章简单描述，会展示在主页
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

## c 语言结构体成员和位域在内存中的存储方式

在小端存储的机器上，C编译器对结构体的成员按它们的声明顺序从低到高的地址进行存储，先定义的成员存储在内存的低地址，使用位域的时候也是类似，先定义的成员先占用低比特位。


## c 语言怎么描述硬件寄存器？

通常硬件寄存器会映射到一段特定的内存地址，我们可以通过指针来访问和修改寄存器的内容。每个寄存器的功能不一定相同，需要根据硬件手册来具体定义。下面以某个芯片平台的 pwm 控制寄存器为例，说明如何使用 c 语言描述硬件寄存器。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-20230719011111212.5buks5rpf3c0.webp)

这个平台有 4 个 pwm 外设，控制寄存器的基址分别是 0xD401A000，0xD401A400，0xD401A800 ，0xD401AC00，然后每个 pwm 有三个 32 位 的寄存器，分别是 PWM_CRx，PWM_DCR，PWM_PCR。在这三个寄存器里面，不同的 bit 范围之间又控制着不同的功能。在 c 语言里面，可通过联合体、结构体和位域来定义这些寄存器的功能。

三个 pwm 控制寄存器的基地址:

```c
#include <stdint.h>
#define  PWM0_BASE (0xD401A000)
#define  PWM1_BASE (0xD401A400)
#define  PWM2_BASE (0xD401A800)
#define  PWM3_BASE (0xD401AC00)
```

每个 pwm 有三个寄存器， PWM_CRx，PWM_DCR，PWM_PCR，与偏移地址对应，4字节递增:

```c
typedef volatile struct 
{
    uint32_t PWM_CRx;   /*Offset: 0x00*/
    uint32_t PWM_DCR ;  /*Offset: 0x04*/
    uint32_t PWM_PCR ;  /*Offset: 0x08*/
}PWM_HW_T;
```

将 pwm 寄存器地址转换为 PWM_HW_T * 类型的指针，方便通过指针对数据进行读写:

```c
#define HW_PWM0 ((PWM_HW_T *)(PWM0_BASE))
#define HW_PWM1 ((PWM_HW_T *)(PWM1_BASE))
#define HW_PWM2 ((PWM_HW_T *)(PWM2_BASE))
#define HW_PWM3 ((PWM_HW_T *)(PWM3_BASE))
```

每个寄存器的功能定义:

```c
// PWM_CRx 寄存器定义
typedef union 
{
    uint32_t v;
    struct 
    {
        uint32_t PRESCALE : 6;  //[5:0]
        uint32_t SD : 1;        //[6]
        uint32_t Reserved :25;  //[31:7]
    }d;
}REG_PWM_CRx_T;

// PWM_DCR 寄存器定义
typedef union 
{
    uint32_t v;
    struct 
    {
        uint32_t DCYCLE : 10;   //[9:0]
        uint32_t FD :1;         //[10]
        uint32_t Reserved :21;  //[31:11]
    }d;
}REG_PWM_DCR_T;

// PWM_PCR 寄存器定义
typedef union 
{
    uint32_t v;
    struct 
    {
        uint32_t PV : 10;     	//[9:0]
        uint32_t Reserved :22;  //[31:10]
    }d;
}REG_PWM_PCR_T;
```

操作 pwm 寄存器示例，读和写寄存器都可以通过指针操作实现:

```c
void PWM0_Init(void)
{
    PWM_HW_T *PWM = HW_PWM0;
    REG_PWM_CRx_T PWM_CRx = {0};
    REG_PWM_DCR_T PWM_DCR = {0};
    REG_PWM_PCR_T PWM_PCR = {0};


    PWM_CRx.d.PRESCALE = 0x3F;
    PWM_CRx.d.SD = 0;
    PWM_CRx.d.Reserved = 0;
    // 写  pwm0 的 PWM_CRx 寄存器
    PWM->PWM_CRx = PWM_CRx.v;
	
    // 读 pwm0 的 PWM_DCR 寄存器
    PWM_DCR.v = PWM->PWM_DCR;

    PWM_PCR.d.PV = 0x3FF;
    PWM_PCR.d.Reserved = 0;
    PWM->PWM_PCR = PWM_PCR.v; 
}
```

