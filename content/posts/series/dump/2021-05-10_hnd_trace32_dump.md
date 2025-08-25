---
title: "用 Trace32 分析死机问题"
date: 2021-05-10T13:28:48+08:00
lastmod: 2023-03-18T13:28:48+08:00
author: ["hacper"]
tags:
    - dump
    - trace32
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
# cover:
#     image: ""
#     caption: ""
#     alt: ""
#     relative: false

---

出现死机问题的设备是展锐8910，打开Trace32软件，导入设备死机时的dump文件进行分析。如下图：

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled.6pc338h1s6c0.png)

先看死机时PC停止的位置，对应的汇编代码是 ldrb r2,[r1,#0x5]，结合trace32中寄存器的值来看，可以确定死机原因是访问非法内存， r1 寄存器为0，也就是访问0地址（空指针）导致死机。



下面分析出现访问空指针问题的具体原因。

先分析当前的函数调用栈，为了方便查看，把调用栈信息复制出来，如下所示：

```c
ql_open_file_get_info(
    position = ?,
    file_info = 0x80CD343C)
  err = 0
  node = 0x0
  __FUNCTION__ = (113, 108, 95, 111, 112, 101, 110, 95, 102, 105, 108, 101, 95, 103, 101, 116, 95, 1

ql_file_is_opened(
    file_name = 0x80CD3614)
  cnt = 8
  err = -2134035396
  position = 7
  file_info = (
    file_name = (85, 70, 83, 58, 106, 115, 111, 110, 46, 116, 120, 116, 0, 79, 189, 128, 12, 0, 0, 0
    file_open_mode = 522,
    fd = 1033)

ql_fopen(
    file_name = 0x80CD3614,
    mode = 0x6032500C)
  file_real_path = (47, 117, 115, 101, 114, 47, 47, 99, 111, 114, 103, 101, 116, 47, 99, 111, 110, 1
  file_type = 0
  open_mode = 1537
  file_info = (
    file_name = (12, 0, 0, 0, 122, 46, 49, 96, 52, 0, 0, 0, 176, 70, 49, 96, 16, 0, 0, 0, 80, 53, 20
    file_open_mode = 970205137,
    fd = -2134034952)
  err = -2134035396
  __FUNCTION__ = (113, 108, 95, 102, 111, 112, 101, 110, 0)

OEMFile_Open(
    path = 0x6032D8A8,
    oflag = 0x6032500C)
  fd = -2134035396
  _path = (85, 70, 83, 58, 47, 99, 111, 114, 103, 101, 116, 47, 99, 111, 110, 102, 105, 103, 46, 116
  __FUNCTION__ = (79, 69, 77, 70, 105, 108, 101, 95, 79, 112, 101, 110, 0)

    }
    else
CEMConfig::SaveConfig(
    this = 0x80CD4578)
  pFile = 0x80CD343C
  buf = (168, 46, 167, 128, 224, 33, 158, 128, 196, 32, 154, 128, 221, 214, 33, 96, 4, 0, 0, 0, 248,

CSystemEngine::SaveConfig(
    this = 0x80CD48A8)
  buffer = (52, 50, 57, 52, 57, 54, 55, 50, 57, 53, 0, 128, 140, 148, 220, 128, 68, 122, 220, 128, 1

CPocUIManager::HandleATCommand(
    this = 0x80CD3C50,
    aData = 0x80CD42AC,
    aLength = 4)
  mtd = 124

CPocUIManager::HandleATMessage(
    this = 0x80CD3C50,
    pMsg = 0x80C3EAA8)
  param_length = 4
  res = -2134035396

OEM_PendMessage(
    pmsg = 0x80A72D18)
  msg = 0x80A72D18

ql_corget_api_task_worker(
  ?)
  recv_msg = (id = MSG_ID_POC, payload = 0x80A72D18)
  ret = -2134035396
  __FUNCTION__ = (113, 108, 95, 99, 111, 114, 103, 101, 116, 95, 97, 112, 105, 95, 116, 97, 115, 107

    g_pcm_play_ctx.pcm_play_handle = ql_pcm_open(&config, flag);
}
prvTaskExitError()

end of frame
```

可以看到，在Trace32 里面不仅可以看到当前出现问题时的调用栈关系，还可以看到函数参数，变量的具体值，对于分析问题的来龙去脉非常有利。

从调用栈看，当前正在调用接口ql_fopen( file_name = 0x80CD3614, mode = 0x6032500C)打开文件，输入查看变量的指令来查看地址 0x80CD3614，0x6032500C的数据：

```
v.v %s (char *)0x80CD3614
v.v %s (char *)0x6032500C
```

可以看到当前打开的文件是"UFS:/corget/config.txt"，打开方式为 "wb"。
{{< figure link="https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled 1.78fnrbowyhs0.png" >}}

结合源码和调用栈来看，ql_fopen内部调用ql_file_is_opened(file_name = 0x80CD3614) 检查UFS:/corget/config.txt是否在已打开的文件列表内。
{{< figure link="https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled 2.37ocqw9ddjg0.png" >}}

然后ql_file_is_opened 内部会调用 ql_open_file_get_info 遍历打开的文件信息列表，查找文件是否已打开，从dump和源码可以知道，当前总共打开了8个文件，ql_file_is_opened 中 position = 7，也就是当前在获取最后一个文件的信息（从0开始，0~7）。
{{< figure link="https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled 3.5l1t54o27l00.png" >}}

最终在ql_open_file_get_info里访问了空指针死机，ql_open_file_get_info里面

```
err = 0
node = 0x0 -> NULL
```
{{< figure link="https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled 4.14pnhuh5bcik.png" >}}

根据代码可以看到是ql_open_file_get_info里面memcpy(file_info, node→data, node→size)这里访问了空指针，可以结合dump和linklist的数据结构定义交叉验证: 

```c
typedef struct node
{
	struct node * next; // 4 bytes
    datasize_t size; // 1 bytes
    data_t data[1]; // 4 bytes
} linklist; // 9 bytes
```

{{< figure link="https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled 5.2rjbg4i36ga0.png" >}}
死机的汇编代码位置 ldrb r2,[r1,#0x5]，r1为0，偏移5个字节，刚好是memcpy(file_info, node→data, node→size)中的node→data。

为什么ql_open_file_get_info里会访问空指针？其实是linklist_node_find 这个函数的bug，查看全局变量ql_file_open_list的值和linklist_node_find的源码，可以定位到问题的原因是linklist_node_find的代码逻辑问题，查找链表节点中遍历链表是从(*linklist_handler)->header->next;开始，忽略了第一个节点，如果当前查找的节点是链表的最后一个节点，就会返回空节点。

linklist_node_find函数的源码如下：

```c
QuecOSStatus linklist_node_find(linklist_handler_t * linklist_handler, uint32_t i, linklist * * dest_node)
{
	int 		 j = 0;

    if(! linklist_handler)
    {
		return kGeneralErr;
    }

    if(! *linklist_handler)
    {
   	    return kGeneralErr;
    }

	if(! dest_node)
	{
		return kGeneralErr;
	}

    linklist * temp_node = (*linklist_handler)->header->next;

    if(i >= (*linklist_handler)->node_cnt)
    {
   		return kGeneralErr;
    }

    for(j=0; j<=i; j++)
    {
        if(j == i)
        {
            *dest_node = temp_node;
            break;
        }
        temp_node = temp_node->next;
    }
	return kNoErr;
}
```

{{< figure link="https://github.com/hacperme/picx_hosting/raw/master/20210507/Untitled 6.6ei9ltmtxpo0.png" >}}

通过Trace32分析dump文件，定位到了出现问题的源码位置，这个死机问题基本上完成了追根溯源，那么对于如何解决这个问题，你有什么建议？欢迎讨论。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/qrcode_for_gh_b1444a13ac67_258.4g56jp6fs4y0.jpg)