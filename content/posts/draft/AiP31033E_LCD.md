---
title: "AiP31033E 段码屏调试总结"
date: 2023-03-05T18:28:48+08:00
lastmod: 2023-03-05T18:28:48+08:00
author: ["hacper"]
tags:
    - AiP31033E
    - LCD
    - spi
categories:
    - 笔记
draft: true

description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
mermaid: true
---

## 硬件连接



| Pin Num | Pin Name     | Net Name      | MUX NUM | GPIO NUM | I/O  | Part |
| ------- | ------------ | ------------- | ------- | -------- | ---- | ---- |
| 64      | LCD_RST      | LCD_RST       | 2       | GPIO_41  | DO   | LCD  |
| 65      | LCD_SPI_CS   | LCD_SPI_CS    | 2       | GPIO_38  | DO   |      |
| 66      | LCD_SPI_DOUT | LCD_SPI_DOUT  | 2       | GPIO_35  | DIO  |      |
| 67      | LCD_SPI_CLK  | LCD_SPI_CLK   | 2       | GPIO_37  | DO   |      |
| 134     | LCD_VDDIO    | LCD_VDDIO_3V2 |         |          | PO   |      |
| 135     | LCD_ISINK    | LCD_ISINK     |         |          | PI   |      |

信号线只有：LCD_SPI_CS   LCD_SPI_DOUT LCD_SPI_CLK，是三线spi接口的屏幕。

屏幕驱动 IC 是 AiP31033E。

## 驱动

尝试移植到模组的LCM 接口未成功，改用 GPIO 模拟SPI的方式驱动屏幕。

屏幕驱动的主要内容

1. 引脚初始化，将LCD的信号脚复用为gpio功能，并初始化

   ```c
   ql_gpio_init(AIP_SPI_RES, GPIO_OUTPUT, PULL_NONE, LVL_LOW); // RST
   ql_gpio_init(AIP_SPI_CS, GPIO_OUTPUT, PULL_NONE, LVL_LOW);  // CS
   ql_gpio_init(AIP_SPI_DIO, GPIO_OUTPUT, PULL_NONE, LVL_LOW); // DIO
   ql_gpio_init(AIP_SPI_CLK, GPIO_OUTPUT, PULL_NONE, LVL_LOW); // CLK
   ```

2. 屏幕复位

   ![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230303193117129-1678012392567.5yeiepfpb2o0.png)

   ```c
   ql_gpio_set_level(AIP_SPI_RES, LVL_HIGH);
   ql_delay_us(5);
   ql_gpio_set_level(AIP_SPI_RES, LVL_LOW);
   ql_delay_us(5);
   ql_gpio_set_level(AIP_SPI_RES, LVL_HIGH);
   ql_delay_us(50);
   ```

3. gpio 模拟spi 通信，只发数据

   写命令

   ```c
   static void write_com(int para)
   {
       int j;
       j = 8;
       ql_gpio_set_level(AIP_SPI_CS, LVL_LOW);
   
       ql_gpio_set_level(AIP_SPI_CLK, LVL_LOW);
   
       ql_gpio_set_level(AIP_SPI_DIO, LVL_LOW);
   
       ql_gpio_set_level(AIP_SPI_CLK, LVL_HIGH);
   
       do
       {
           if (para & 0x80)
               ql_gpio_set_level(AIP_SPI_DIO, LVL_HIGH);
           else
               ql_gpio_set_level(AIP_SPI_DIO, LVL_LOW);
   
           ql_gpio_set_level(AIP_SPI_CLK, LVL_LOW);
   
           ql_gpio_set_level(AIP_SPI_CLK, LVL_HIGH);
           --j;
           para <<= 1;
       } while (j);
   
       ql_gpio_set_level(AIP_SPI_CS, LVL_HIGH);
   }
   ```

   

   写数据

   ```c
   static void write_data(int para)
   {
       int j;
       j = 8;
       ql_gpio_set_level(AIP_SPI_CS, LVL_LOW);
       ql_gpio_set_level(AIP_SPI_CLK, LVL_LOW);
       ql_gpio_set_level(AIP_SPI_DIO, LVL_HIGH);
       ql_gpio_set_level(AIP_SPI_CLK, LVL_HIGH);
       do
       {
           if (para & 0x80)
               ql_gpio_set_level(AIP_SPI_DIO, LVL_HIGH);
           else
               ql_gpio_set_level(AIP_SPI_DIO, LVL_LOW);
           ql_gpio_set_level(AIP_SPI_CLK, LVL_LOW);
   
           ql_gpio_set_level(AIP_SPI_CLK, LVL_HIGH);
           --j;
           para <<= 1;
       } while (j);
       ql_gpio_set_level(AIP_SPI_CS, LVL_HIGH);
   }
   ```

   三线spi 实际是发 9bit 数据，发送的是命令还是数据，嵌入到数据流之中。

   三线spi接口与4线spi接口的区别，4线spi多了一个数据命令信号脚，通过这个脚拉高拉低来区分发送的是命令还是数据。

   ![image-20230303195143919](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230303195143919.wjn128em0qo.png)

   

   ![image-20230303195214629](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230303195214629.4n0thy27f9g0.png)

   

4. 屏幕参数初始

   用供应商提供的初始化参数，具体含义可以对照芯片手册

   ```c
   write_com(0xe2); // software reset
   write_com(0x2f); // power control
   write_com(0xa0); // MX=0
   write_com(0xc0); // MY=0
   write_com(0xa4); //
   write_com(0xf1); //
   write_com(0x60); //
   write_com(0xac); //
   //	write_com(0x70);
   write_com(0xf0); //
   //	write_com(0x80);
   write_com(0xaF); // display on
   ```

5. 刷屏

   先写地址，后写数据

   ![image-20230303205453013](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230303205453013.21ij5i9jgh0g.png)

   ```c
   // 写ddram 到屏幕显示
   int Hal_lcd_reflash(void)
   {
       uint8_t *p = NULL;
       ql_rtos_mutex_lock(g_lcd.lock, QL_WAIT_FOREVER);
       write_com(0xB0); // set page address
       write_com(0x10); // set column address high 3bit
       write_com(0x00); // set column address low 4bit
       p = &g_lcd.ddr_ram[0];
       for (int i = 0; i < sizeof(g_lcd.ddr_ram); i++)
       {
           write_data(*p);
           p++;
       }
   
       ql_rtos_mutex_unlock(g_lcd.lock);
   
       return 0;
   }
   ```

   控制芯片有一个 4*96 bit 的ddram, 用于存放显示数据，要显示什么内容，就根据逻辑表填写好ddrm的数据，发送到屏幕去刷新显示。

## 逻辑表

![image-20230303205942238](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230303205942238.323e0y32zxo0.png)

逻辑表与ddrm的对应关系：

从逻辑表看，这个屏幕有32个seg，4个com。ddram 可以定义一个32字节的数组缓存显示数据，显示数据填好之后就可以写入到屏幕进行刷新显示。

ddrm 中的每一个bit位置0、置1，控制液晶屏中的某一段的亮灭。

比如: 最近收款 是T7， com= 3, seg= 1，T7的显示控制在ddram数组中的 ddram[1]元素的第3bit控制：

点亮T7  ddram[1] |= 1 <<3;

清除T7  ddram[1] &=  ~（1<<3）



弄清楚逻辑表与ddram的对应关系之后，可以将这种关系写到代码里。

怎么去描述呢？

需要知道每个段在ddram中的位置，根据逻辑表来确定，在代码上也建立一个表来记录这些段在ddram的位置。



```c
typedef struct
{
    uint8_t com; // COM 在逻辑表的位置
    uint8_t seg; // SEG 在逻辑表的位置
}ds_logic_t;

// 段码屏逻辑表定义
typedef enum
{
    DS_FONT_LEIJI_T1 = 0, // 累计 T1
    DS_FONT_YESTODAY_T2,  // 昨天 T2
    DS_FONT_TODAY_T3,     // 今天 T3
    DS_FONT_WEEK_T4,      // 本周 T4
    DS_FONT_MONTH_T5,     // 本月 T5
    DS_FONT_YUAN_T6,      // 元 T6
    DS_FONT_RECENT_T7,    // 最近收款 T7
    DS_FONT_SEC_T8,       // 今天第几笔 T8
    DS_FONT_RECV_TIME_T9,    // 收款时间 T9
    DS_FONT_COL1,         // COL1
    DS_FONT_DOT,          // DOT
    DS_FONT_COL2,         // COL2
    DS_FONT_MAX
} ds_font_e;


typedef enum
{
    DS_SIGNAL_S1 = 0, // 无网络 S1
    DS_SIGNAL_S2,     // 1 格信号 S2
    DS_SIGNAL_S3,
    DS_SIGNAL_S4,
    DS_SIGNAL_MAX
}ds_signal_level_e;


typedef enum
{
    DS_BATTARY_H1 = 0,  // 电池电量空 H1
    DS_BATTARY_H4,      // 电池电量1格 H4 
    DS_BATTARY_H3,
    DS_BATTARY_H2,
    DS_BATTARY_MAX
}ds_battary_e;

typedef enum
{
    DS_NUM_1 = 0, // 第一位数字
    DS_NUM_2,
    DS_NUM_3,
    DS_NUM_4,
    DS_NUM_5,
    DS_NUM_6,
    DS_NUM_7,
    DS_NUM_8,
    DS_NUM_9,
    DS_NUM_10,
    DS_NUM_11,
    DS_NUM_12,
    DS_NUM_13,
    DS_NUM_14,
    DS_NUM_15,
    DS_NUM_MAX
}ds_num_e;


// 15个数字，每个数字 ABCDEFG 段
typedef struct 
{
    ds_logic_t a;
    ds_logic_t b;
    ds_logic_t c;
    ds_logic_t d;
    ds_logic_t e;
    ds_logic_t f;
    ds_logic_t g;
}ds_logic_num_t;

// 按abcdefg顺序排列，对应每一个bit位
static uint8_t num_font[] = {
    /*0*/0x3f,
    /*1*/0x06,
    /*2*/0x5b,
    /*3*/0x4f,
    /*4*/0x66,
    /*5*/0x6d,
    /*6*/0x7d,
    /*7*/0x07,
    /*8*/0x7f,
    /*9*/0x6f,
    /*-*/0x40,
    /*off*/0x00,
};

static ds_logic_num_t num_logic[DS_NUM_MAX] = {
    [DS_NUM_1] = {  .a={.com = 0, .seg = 31, }, 
                    .b={.com = 1, .seg = 30, }, 
                    .c={.com = 3, .seg = 30, }, 
                    .d={.com = 3, .seg = 31, }, 
                    .e={.com = 2, .seg = 31, }, 
                    .f={.com = 1, .seg = 31, }, 
                    .g={.com = 2, .seg = 30, }, },

    [DS_NUM_2] = {  .a={.com = 0, .seg = 29, }, 
                    .b={.com = 1, .seg = 28, }, 
                    .c={.com = 3, .seg = 28, }, 
                    .d={.com = 3, .seg = 29, }, 
                    .e={.com = 2, .seg = 29, }, 
                    .f={.com = 1, .seg = 29, }, 
                    .g={.com = 2, .seg = 28, }, },
        
};

static ds_logic_t bat_level[DS_BATTARY_MAX] =
    {
        [DS_BATTARY_H1] = {
            .com = 3,
            .seg = 14,

        },
        [DS_BATTARY_H2] = {
            .com = 2,
            .seg = 14,

        },
        [DS_BATTARY_H3] = {
            .com = 1,
            .seg = 14,

        },
        [DS_BATTARY_H4] = {
            .com = 0,
            .seg = 14,

        },

};

static ds_logic_t signal_level[DS_SIGNAL_MAX] = {
    [DS_SIGNAL_S1] = {
        .com = 0,
        .seg = 18,

    },
    [DS_SIGNAL_S2] = {
        .com = 1,
        .seg = 15,

    },
    [DS_SIGNAL_S3] = {
        .com = 2,
        .seg = 15,

    },
    [DS_SIGNAL_S4] = {
        .com = 3,
        .seg = 15,

    },
};

static ds_logic_t font_logic[DS_FONT_MAX] = {
    [DS_FONT_LEIJI_T1] = {
        .com = 0,
        .seg = 30,

    },
    [DS_FONT_YESTODAY_T2] = {
        .com = 0,
        .seg = 28,

    },
    [DS_FONT_TODAY_T3] = {
        .com = 0,
        .seg = 26,

    },
    [DS_FONT_WEEK_T4] = {
        .com = 0,
        .seg = 22,

    },
    [DS_FONT_MONTH_T5] = {
        .com = 0,
        .seg = 20,

    },
    [DS_FONT_YUAN_T6] = {
        .com = 0,
        .seg = 16,

    },
    [DS_FONT_RECENT_T7] = {
        .com = 3,
        .seg = 1,

    },
    [DS_FONT_SEC_T8] = {
        .com = 3,
        .seg = 5,

    },
    [DS_FONT_RECV_TIME_T9] = {
        .com = 3,
        .seg = 7,

    },
    [DS_FONT_COL1] = {
        .com = 0,
        .seg = 24,

    },
    [DS_FONT_DOT] = {
        .com = 3,
        .seg = 11,

    },
    [DS_FONT_COL2] = {
        .com = 3,
        .seg = 9,

    },
};

```



再定义一些接口来显示这些元素



```c
int Hal_lcd_display_battary_level(int off, int level)
{
    ql_rtos_mutex_lock(g_lcd.lock, QL_WAIT_FOREVER);
    if (off == 1) // 关闭显示
    {
        for (size_t i = 0; i < DS_BATTARY_MAX; i++)
        {
            g_lcd.ddr_ram[bat_level[i].seg] = 0;
        }
    }
    else
    {
        if (level < DS_BATTARY_MAX)
        {
            for (size_t i = 0; i < DS_BATTARY_MAX; i++)
            {
                if (i <= level)
                {
                    g_lcd.ddr_ram[bat_level[i].seg] |= (1 << bat_level[i].com);
                }
                else
                {
                    g_lcd.ddr_ram[bat_level[i].seg] &= (~(1 << bat_level[i].com));
                }
            }
        }
        else
        {
            Log_e("invaild level%d", level);
        }
    }

    ql_rtos_mutex_unlock(g_lcd.lock);

    return 0;
}


int Hal_lcd_display_signal_level(int off, int level)
{
    ql_rtos_mutex_lock(g_lcd.lock, QL_WAIT_FOREVER);
    if (off == 1) // 关闭显示
    {
        for (size_t i = 0; i < DS_SIGNAL_MAX; i++)
        {
            g_lcd.ddr_ram[signal_level[i].seg] = 0;
        }
    }
    else
    {

        switch (level)
        {
        case DS_SIGNAL_S1:
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S1].seg] |= (1 << signal_level[DS_SIGNAL_S1].com);
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S2].seg] &= (~(1 << signal_level[DS_SIGNAL_S2].com));
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S3].seg] &= (~(1 << signal_level[DS_SIGNAL_S3].com));
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S4].seg] &= (~(1 << signal_level[DS_SIGNAL_S4].com));
            break;
        case DS_SIGNAL_S2:
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S1].seg] &= (~(1 << signal_level[DS_SIGNAL_S1].com));
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S2].seg] |= (1 << signal_level[DS_SIGNAL_S2].com);
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S3].seg] &= (~(1 << signal_level[DS_SIGNAL_S3].com));
            g_lcd.ddr_ram[signal_level[DS_SIGNAL_S4].seg] &= (~(1 << signal_level[DS_SIGNAL_S4].com));
            break;
   default:
            Log_e("invaild level%d", level);
            break;
        }
    }

    ql_rtos_mutex_unlock(g_lcd.lock);

    return 0;
}


int Hal_lcd_display_font(ds_font_e font, int on)
{
    ql_rtos_mutex_lock(g_lcd.lock, QL_WAIT_FOREVER);
    if (on)
    {
        g_lcd.ddr_ram[font_logic[font].seg] |= (1 << font_logic[font].com);
    }
    else
    {
        g_lcd.ddr_ram[font_logic[font].seg] &= (~(1 << font_logic[font].com));
    }

    ql_rtos_mutex_unlock(g_lcd.lock);

    return 0;
}


```

## 界面绘制

