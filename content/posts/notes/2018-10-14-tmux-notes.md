---
id: 1702
title: 'tmux 基础教程'
date: '2018-10-14T06:43:58+08:00'
author: hacper
excerpt: 'tmux 是一个非常好用的终端复用软件（Terminal Multiplexer），在操作远程服务器时可以用它来保持会话，避免突然断线的尴尬。也可以用它来实现多分屏，提高工作效率。'
layout: post
guid: 'https://hacperme.com/?p=1702'
permalink: /2018/10/14/tmux-notes/
Steempress_sp_steem_publish:
    - '0'
views:
    - '1477'
categories:
    - 笔记
tags:
    - Tmux
---

![blob.jpg](https://i.loli.net/2018/10/09/5bbc95317ff9d.jpg)

tmux 是一个非常好用的终端复用软件（Terminal Multiplexer），在操作远程服务器时可以用它来保持会话，避免突然断线的尴尬。也可以用它来实现分屏多任务，提高工作效率。

## 安装 tmux

我使用的操作系统是Ubuntu，这里以常规安装软件的方式为例安装 tmux。其他操作系统和安装方式可以网上找找，应该都不难。

在 Ubuntu 系统使用下面的命令安装 tmux：

`sudo apt install tmux -y`

![blob.jpg](https://i.loli.net/2018/10/09/5bbc967240edf.jpg)

在使用 tmux 之前，先通过下图了解一下它的 服务器，会话，窗口，面板之间的包含关系，有助于理解 tmux 的工作方式。

![blob.jpg](https://i.loli.net/2018/10/09/5bbc9bd47dd10.jpg)

[image source](https://leanpub.com/the-tao-of-tmux/read#leanpub-auto-window-manager-for-the-terminal)

## 与会话有关的操作命令

- 创建新会话，s1是自定义的会话名称  
  `tmux new -s s1`
- 返回原来的终端，会话保持在后台运行  
  按ctrl+b 在按d键  
  `ctrl+b d`

ctrl+b 是tmux默认的命令前缀。

- 列出会话列表  
  `tmux ls`

![blob.jpg](https://i.loli.net/2018/10/09/5bbca1223f141.jpg)

- 进入会话  
  -t参数后面加上会话的名称，比如：下面的 s1  
  `tmux a -t s1`
- 在当前会话中查看和切换会话  
  通过方向键选择切换会话  
  `ctrl+b s`

![blob.jpg](https://i.loli.net/2018/10/09/5bbca21a4ce8f.jpg)

- 终止会话  
  在会话外：  
  `tmux kill-session -t s2`  
  在会话中：  
  `ctrl+b : kill-session -t s1`
- 重命名会话  
  在会话内和在会话外，同样有两种方式。  
  `tmux rename -t s2 s1`  
  `ctrl + b $ s2`

其实可以输入 `ctrl + b ？` 查看会话中的所有命令  
![blob.jpg](https://i.loli.net/2018/10/09/5bbca6499d809.jpg)

## 与窗口有关的操作命令

- 修改当前窗口的名字  
  crtl+b 再输入一个逗号 加名字，比如将当前窗口重命名为w1:  
  `crtl+b , w1`

![blob.jpg](https://i.loli.net/2018/10/09/5bbca7d5f206d.jpg)

- 创建新窗口  
  `ctrl+b c`

将新窗口命名为：w2  
当前窗口用\*符号作为标记

![blob.jpg](https://i.loli.net/2018/10/09/5bbca8c4ad1eb.jpg)

- 切换window  
  ctrl+b 后面输入窗口编号  
  `ctrl+b 0`
- 关闭窗口  
  `ctrl+b &`

## 与面板有关的操作命令

- 垂直分屏  
  `ctrl+b %`

![blob.jpg](https://i.loli.net/2018/10/09/5bbc84fce78c2.jpg)

- 水平分屏  
  `ctrl+b "`

![blob.jpg](https://i.loli.net/2018/10/09/5bbc85a4cba75.jpg)

这样便在一个窗口创建了三个工作面板

- 切换面板  
  `ctrl+b 再输入方向键`
- 调节面板大小  
  `ctrl+b 按住 Alt键 再输入方向键`
- 关闭面板  
  `ctrl+b x`

- - - - - -

参考：

- [The Tao of tmux](https://leanpub.com/the-tao-of-tmux/read#leanpub-auto-window-manager-for-the-terminal)
- [tmux简明快速教程](http://blog.kissdata.com/2014/07/29/tmux.html)