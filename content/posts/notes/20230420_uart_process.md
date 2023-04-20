---
title: "串口数据接收分包、粘包问题的处理方法"
date: 2023-04-20T21:42:17+08:00
lastmod: 2023-04-20T21:42:17+08:00
author: ["hacper"]
tags:
    - 串口
    - 分包
    - 粘包
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "一种串口数据分包粘包问题的处理方法" # 文章简单描述，会展示在主页
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



## 介绍

串口是一种常见的通信外设，广泛应用于各种设备之间的数据通信。在串口通信中，设备之间需要约定通信协议，以确保数据的正确传输和解析。常见的串口协议由帧头、数据长度、帧类型、数据、校验等字段构成，这类协议设计较为完善，可以在接收的字节流中按照协议格式正确地提取一帧数据，对于出现粘包或者分包的情况也能正确解析。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image-20230420232407103.48m3zxzs7v80.webp)

然而，并不是所有的串口协议都设计得那么完善。有些串口协议可能没有用于同步的数据帧头和帧尾，如果一帧数据出现了分包（分多次接收），那么将无法解析出正确的数据。这种情况下，需要采取一些特殊的处理方法，以确保数据的正确解析和传输。

## 解决方法

在串口通信中，数据的分包和粘包问题是比较常见的，会影响数据的正确解析和传输。为了解决这个问题，可以采用一些有效的处理方法。

一种常见的处理方法是加入帧超时机制。在串口接收数据时，不断地检测两个字节或者两次接收数据的间隔时间，当间隔时间超过某一值，则认为一帧数据接收完成，一帧数据接收完整之后再做数据解析和具体业务。默认的超时时间间隔可以设置为 50 ms，然后根据实际情况稍微调大或者调小超时时间间隔。这种方法可以有效地解决数据分包和粘包问题，但也会带来串口接收性能下降的问题。

处理逻辑可以参考下面的伪代码：

```c
void UartInit(void)
{
    // 初始化串口硬件参数
    
    // 创建消息队列和ringbuff
    ret = rtos_init_queue(&uart_queue, "uart_q", sizeof(uart_msg_t), 20);
    rb_init(&g_uart_rb, g_uart1_rx_buff, sizeof(g_uart1_rx_buff));

    // 创建一个定时器 50ms
    ret = rtos_init_timer(&uart_timer, 50, uart_timer_cb, 0);
	
    // 创建串口接收处理线程
    ret = rtos_create_thread(&uart_main_thread,
                            20,
                            "uart_main",
                            uart_main_thread_entry,
                            4 * 1024,
                            NULL);

    // 注册串口中断回调
    uart_set_rx_callback(port, uart_rx_cb);

}

// 串口接收中断回调
static void uart_rx_cb(int uport, void *param)
{
    uart_msg_t send_msg = {0};
    send_msg.type = 0;
    //有串口数据进来，发消息通知串口数据处理线程去读取串口数据
    if (uart_queue)
        rtos_push_to_queue(&uart_queue, &send_msg, 0);
}

// 串口数据接收处理任务
static void uart_main_thread_entry(void *arg)
{
    OSStatus err = kNoErr;
    int ret;
    uint32_t read_len = 0, recv_len = 0, free_len=0;
    uart_msg_t recv_msg = {0};
    while (1)
    {
        err = rtos_pop_from_queue(&uart_queue, &recv_msg, 100);
        if (err != kNoErr)
        {
            continue;
        }
        switch (recv_msg.type)
        {
        case 0:
            //收到串口有数据消息。
            //读取串口数据，写入到ringbuffer中先缓存，同时重新启动帧超时定时器
            break;
        case 1:
            // 收到定时器超时消息
            // 一帧数据接收完成，进行数据解析和业务处理
            break;
        default:
            break;
        }
    }
}

// 定时器超时回调
static void uart_timer_cb( void *arg )
{
    uart_msg_t send_msg = {0};
    rtos_stop_timer(&uart_timer);
    send_msg.type = 1;
    // 发消息通知处理线程，完成一帧数据接收
    if (uart_queue)
        rtos_push_to_queue(&uart_queue, &send_msg, 0);
}
```

引入帧超时机制虽然解决了串口数据分包问题，但也带来了串口接收性能下降的问题。如果串口分包数据大小是固定的，比如每次 20 字节分包一次，那么还可以稍微优化一下接收速度，如果收到的数据不是固定分包字节的倍数，也可以认为一帧数据接收结束了，不必再去等定时器超时，这种方法可以优化串口接收数据的性能。

另外，如果多次连续发送串口数据，有可能会出现粘包现象，如果粘包对业务有影响，那么需要要求串口发送两帧数据之间的间隔要大于一定时间。最好是要求串口发送字节之间小于一定时间，比如 10 ms，发送数据帧之间也大于一定时间，比如 100 ms，这样分包和粘包的问题都得以解决。

## 总结

在设计串口协议时，需要考虑到数据的同步、长度和校验等因素，确保数据能够正确解析和传输。没有数据同步标志的串口协议，可以加入帧超时机制解决数据分包问题，缺点就是牺牲一点串口接收数据的性能。


