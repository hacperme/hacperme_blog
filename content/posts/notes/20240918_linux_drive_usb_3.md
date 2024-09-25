---
title: "USB 知识笔记（三）"
date: 2024-09-18T01:31:42+08:00
lastmod: 2024-09-18T01:31:42+08:00
author: ["hacper"]
tags:
    - Linux
    - USB
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
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

# USB 描述符

描述符描述了该USB设备所有的属性和可配置信息，如设备所属于的类（Class）、接口（interface）信息、端点（endpoint）信息。标准的USB设备有6种常用的USB描述符：设备描述符、配置描述符、字符串描述符、接口描述符、端点描述符、设备限定描述符。另外，还有一种特殊的描述符称为接口关联描述符，用于将一组有关的描述符关联起来共同描述一个特定的功能。



USB描述符结构框图：



接口0和接口1通过接口关联描述符1关联在一起共同描述一个功能；接口2和接口3通过接口关联描述符2关联在一起共同描述另外一个功能；而接口 4 单独描述其他的一个功能。



设备描述符指明了该设备有几个配置描述符，每个配置描述符都分别指明了该配置描述符中的接口描述符，而接口描述符指明了该接口有几个端点描述符。

在同一个配置描述符中的多个功能（由一个或者多个接口描述符描述）能够同时工作，但是如果USB设备存在多个配置描述符，USB主机会使用其中一个，其他的配置描述符中所描述的功能则不能工作。



## 设备描述符

一个设备有且只有一个设备描述符。在设备描述符中只会给出这个设备所支持的配置描述符（Configuration Descriptor）的数量：bNumConfigurations，设备的配置描述符的索引从1开始，比如当设备的配置描述符有两个时，这两个配置描述符的索引分别是1和2。USB主机就是使用这个索引作为GetDescriptor（Configuration）的参数来分别获取对应的设备的配置描述符的。



![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.7p9dwarty.webp)

## 配置描述符

配置描述符定义了设备的一种配置信息，一个设备可以有一个或者多个配置描述符。配置描述符包括了配置的基本信息，如该设备的接口描述符的个数、配置描述符的长度、供电属性等。当主机需要获取配置描述符时，该配置描述符所拥有的接口描述
符和端点描述符都一并返回。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.6f0ne2b3dv.webp)

配置描述符的长度（wTotalLength）描述了该配置描述符的总长度，即从该描述符的bLength 位域开始一直到最后一个端点描述符结尾为止的总长度，包括该描述符本身的长度，以及配置描述符中所有接口描述符和端点描述符的总长度。

接口数目（bNumInterfaces）表示这个配置有多少个接口描述符，这些接口是控制接口和非控制接口的合集。该配置描述符的配置号信息由 bConfigurationValue 决定，主机就是根据这个值决定在 SetConfiguration（）时给设备选择设置哪个配置描述符，并让设备处于该配置的工作状态。

bmAttributes代表配置的某些属性，第5比特代表该设备是否支持远程唤醒（Remote Wakeup），第6比特代表该设备是否支持自供电。

bMaxPower代表设备在该配置下所消耗的最大电流值，以2mA为单位，如当bMaxPower的值是25时，代表设备在该配置下所消耗的最大电流值为50mA。USB设备多个配置描述符中接口数目不尽相同，bMaxPower也有可能不一样。USB 主机会根据设备在某种配置下 bMaxPower 的值来查看其是否有足够的电流可以提供，最终决定该设备的某种配置是否可用，如果某个设备的所有配置因为USB主机无法提供足够电流而不可用，该设备将无法正常工作。

## 接口描述符

接口描述符位于配置描述符中，指明了某个特殊的USB类的接口。接口描述符通过端点完成数据传输，实现特定类的特定功能。接口描述符通过端点完成数据传输，实现特定类的特定功能。

接口描述符包括该接口的识别号（bInterfaceNumber）、接口的可替换设置号（bAlternateSetting）、端点数目、类和子类、协议等。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.77divt3cac.webp)

接口的替换设置号主要在音频和视频类中使用，一般用于描述不同声道、采样率或者帧率、分辨率的组合。

接口的可替换设置 0 不包含传输端点，其他的可替换设置包含传输端点。之所以要有一个不包含任何传输端点的可替换设置 0，是由 USB音频、视频设备的特性决定的，该设置用于在设备没有被使用时作为设备的默认设置。

当USB设备存在多个可替换设置时，USB主机通过SetInterface（）来指定某个接口的可替换设置（Alternate Setting）。如果 USB 主机没有使用SetInterface（）来指定接口的可替换设置，通信双方默认使用可替换设置0。

端点数目（bNumEndpoints）就是该接口描述符所有支持端点的数目，不包括控制端点0



## 端点描述符

端点描述符包括该端点地址、端点属性、支持的最大包长度、传输时间间隔等。在主机获取配置描述符时，端点描述符和接口描述符
一起返回。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.4qragw2dm5.webp)

端点地址（bEndpointAddress）的第 0～3 比特代表该端点所使用的地址，第7比特代表该端点是用作输入（IN）还是输出（OUT）端点。

端点属性（bmAttributes）的第0和第1比特代表该端点的支持的四种传输属性。如果该端点属于ISO端点，属性的第2～3比特代表该ISO端点的数据同步属于同步、异步、自适应还是无同步类型，属性的第4～5比特代表该ISO端点属于数据传输端点还是反馈端点。

传输时间间隔代表该端点的数据传输间隔了几个帧/微帧，计算方法为2bInterval-1个帧/微帧

## 字符串描述符

字符串描述符是可选的。如果一个设备不支持字符串描述符，需要将设备描述符、配置描述符、接口描述符中的字符串索引值设成
零，字符串描述符用UNICODE编码。

主机获得设备的某个字符串描述符分两条命令：首先主机发送USB标准命令GetDescriptor（），其中所使用的字符串的索引值为0，
设备返回一个零字符串描述符。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.2a521z9tq1.webp)

wLANGID[0]～[x]代表该设备支持的语言，可以从USB设备语言ID规范中获得具体值，典型值0x0409代表英语。

主机根据自己是否支持该语言，再次发出USB标准命令GetDescriptor（），指明所要求得到的字符串的索引值和语言。这次
设备所返回的是 UNICODE编码的字符串描述符。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.9nzraqxpqa.webp)



## 接口关联描述符

对于复合USB设备的接口描述符，可以在每个类（Class）要合并的接口描述符之前加一个接口关联描述符（Interface Association
Descriptor，IAD），其作用就是把多个接口定义成一个类设备。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.m1ihurr3.webp)

bFirstInterface 代表起始的接口编号，bInterfaceCount代表属于这个IAD的接口数目，编号中间不能有间隔。

在一个类的所有合并接口都结束之后，第二个类的所有需要合并的接口又以IAD开始。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.2rv3qizzoo.webp)

IAD1的bFirstInterface为0，bInterfaceCount为2；IAD2的bFirstInterface为2，bInterfaceCount为2。

对于音频类的IAD，音频流接口号必须以一个连续的顺序放在对应的音频控制接口号后面。

## 设备限定描述符

设备限定描述符（Device Qualifier Descriptor）用于描述一个能够同时支持高速和全速模式的USB设备工作在另外一个模式时的设备信息。

例如，当设备工作于全速模式时，设备限定描述符返回它工作于高速模式下的信息。反之，如果设备工作于高速模式时，设备限定描述符返回它工作于全速模式下的信息。

如果一个设备能够同时支持高速和全速模式，并且其在高速模式下和全速模式下信息有所不同，则它必须支持设备限定描述符。

![image](https://github.com/hacperme/picx-images-hosting/raw/master/20240926/image.7w6sfuqp3f.webp)



## 其他速度模式下的配置描述符

其他速度模式下的配置描述符（Other_Speed_Configuration Descriptor）与普通描述符是完全相同的，一般与设备限定描述符一起
使用来描述在其他速度模式下设备的配置信息。
