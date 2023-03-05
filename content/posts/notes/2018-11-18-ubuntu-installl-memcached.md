---
id: 1784
title: '在Linux系统中安装 memcached'
date: '2018-11-18T06:13:47+08:00'
author: hacper
excerpt: '在linux系统中安装 memcached。'
layout: post
guid: 'https://hacperme.com/?p=1784'
permalink: /2018/11/18/ubuntu-installl-memcached/
views:
    - '1270'
categories:
    - 笔记
tags:
    - linux
    - memcached
    - ubuntu
---

![memcached.jpg](https://i.loli.net/2018/11/13/5bea7ede260d6.jpg)

## 安装 memcached

下载最新源码：  
`wget http://memcached.org/latest`  
重命名：  
`mv latest memcached-1.5.10.tar.gz`  
解压：  
`tar -zxvf memcached-1.5.10.tar.gz`  
切换到解压目录：  
`cd memcached-1.5.10/`

编译配置：  
`./configure --prefix=/usr/local/memcached`

出错，缺少 libevent：

```bash
checking for libevent directory... configure: error: libevent is required.  You can get it from http://www.monkey.org/~provos/libevent/

      If it's already installed, specify its path using --with-libevent=/dir/

```

到网站 http://libevent.org/ 下载 libevent 源码。

解压：  
`tar zxvf libevent-2.1.8-stable.tar.gz`  
切换到解压目录：  
`cd libevent-2.1.8-stable`

配置、编译和安装libevent：

```shell
 $ ./configure
 $ make
 $ make verify   # (optional)
 $ sudo make install

```

再回到 memcached 源码的解压目录 memcached-1.5.10

配置：  
`./configure --prefix=/usr/local/memcached`  
编译：  
`make && make test`  
安装：  
`sudo make install`