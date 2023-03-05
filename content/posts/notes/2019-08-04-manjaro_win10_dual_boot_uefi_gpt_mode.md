---
id: 1891
title: '安装 Manjaro 和 Windows 10 双系统 UEFI + GPT 方式'
date: '2019-08-04T17:33:26+08:00'
author: hacper
excerpt: '记录使用 Rufus 制作U盘启动盘和安装 win 10 + manjaro 双系统的步骤。'
layout: post
guid: 'https://hacperme.com/?p=1891'
permalink: /2019/08/04/manjaro_win10_dual_boot_uefi_gpt_mode/
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_publish:
    - '1'
steempress_sp_permlink:
    - manjarowindows10uefigpt-qtv699qb0f
steempress_sp_author:
    - yjcps
views:
    - '8843'
categories:
    - 笔记
tags:
    - manjaro
    - 'Windows 10'
    - 双系统
---

## 工具准备

在工具准备阶段，主要是下载系统镜像文件和制作U盘启动盘的软件。在操作之前请备份好U盘和当前系统上的资料。

- 下载系统镜像文件

对于Windows用户，最新的 Wndows 10 镜像文件可以通过媒体创建工具下载：[官网链接](https://www.microsoft.com/software-download/windows10)，打开这个软件，选择保存为ISO镜像文件就可以了。

非Windows用户不能运行媒体创建工具程序，反而可以直接下载系统镜像文件。在这里下载的镜像文件是：Win10\_1903\_V1\_Chinese(Simplified)\_x64.iso。

![image.png](https://ipfs.busy.org/ipfs/QmXLivVRXtFXYcLhFw3gQdWTctsJu4xoTnaKpmA7676bqM)

Manjaro 镜像文件官方下载地址：[链接](https://manjaro.org/download/)

Manjaro 的下载页面有多个桌面环境选择，这里下载的镜像是官方的XFCE桌面版本，镜像文件为：manjaro-xfce-18.0.4-stable-x86\_64.iso

![image.png](https://ipfs.busy.org/ipfs/QmdeGr2jQL3ntn1gu5y84i1zUGcEBQmcW53FgQyxecbLeh)

- Rufus 制作 U 盘启动盘

Rufus 是一个开源、好用的制作U盘启动盘工具，我一般都使用它来制作系统启动盘，所以安装Windows 10和Manjaro 系统的U盘启动盘都通过Rufus来制作，Rufus的下载地址：[链接](https://rufus.ie/)。

制作Windows10系统的启动盘的设置如下，引导类型选择下载的系统镜像，分区类型设置为 GPT，可以打开日志，其他设置默认。

![image.png](https://ipfs.busy.org/ipfs/QmYvsN7dHEg1erRmt1nt6MSy1jcjMikxMxFnSBr7Q8VU1a)

制作Manjaro系统启动盘的设置和上面一样，但在写入镜像的方式里选择**以DD镜像 模式写入**。

![image.png](https://ipfs.busy.org/ipfs/QmdcRbGB3aJ5Dm9oaT5P2d32UXGR71oc8CzFUKFMp8H6Zz)

## 安装 windows 10

- 设置电脑的启动顺序，优先U盘启动。或者在启动菜单里选择U盘启动，选择带UEFI 的启动项，进入Windows 系统的安装程序。

![image.png](https://ipfs.busy.org/ipfs/QmXY4CieUFWoy1t2HEm6Ldkk8BcNtmdG4sRkVTkxT4MAXu)

- 选择安装专业版

![image.png](https://ipfs.busy.org/ipfs/QmczZCS7Rb1vePd4wwnqLSsnUjCqpww5mEdCCjKVZD8UXi)

- 选择自定义安装

![image.png](https://ipfs.busy.org/ipfs/Qmc7CAZecLcrALP6WE1YUJdnHVv8LKLsBws3wd5LYvRtHW)

- 选择磁盘并新建分区

![image.png](https://ipfs.busy.org/ipfs/QmPhiDY5Qc1z51NtBb84Xi3YmwsC9GKXgZ3vw56vAhtcet)

- 分区大小不要设置为整个磁盘，留有一定空间安装Manjaro

![image.png](https://ipfs.busy.org/ipfs/QmSux4zZQkQTV1J5gkih3uD4ehxG9MY27dxeXN5MjU9LEL)

系统安装位置选择容量较大的那个分区，然后执行下一步进行系统安装。

![image.png](https://ipfs.busy.org/ipfs/QmbygfLnK3kAfVxNkk6QcgfSKzbenmnjwpcxo2y8MPfLgB)

## 安装 Manjaro

- 从U盘启动，进入Manjaro进行系统安装，先设置语言为中文、驱动设置选择Nonefree，**如果是笔记本不选Nonefree的话应该用不了WIFI**，然后进入boot选项进入系统进行安装。

![image.png](https://ipfs.busy.org/ipfs/QmTSGZCMDoPy7Y4s3ebm1tCu1NZDgbCm5gsUMJ3EaAMkYk)

- 之后便进入桌面，点击 install 那个图标进行安装。

![image.png](https://ipfs.busy.org/ipfs/QmT53UxLdfxczxwsgYgGWKfnfszy9HuCYKwTYNjRoT6zwe)

- 选择简体中文，下一步。

![image.png](https://ipfs.busy.org/ipfs/QmWeCaVjquDScdf72xrFocuaKEtg5fBX2JgGvdbrAcZakY)

- 时区设置

![image.png](https://ipfs.busy.org/ipfs/QmNpHnenF9k5KrbiT9GgyBMBaGvM6RYH4v2e2ifBimcqxL)

- 键盘设置，选择汉语拼音

![image.png](https://ipfs.busy.org/ipfs/QmTxKGm7MA9cP5FhJgzfaVZkRZdYJBBraVUR9nDsrhruU5)

- 分区设置，选择手动，在空闲空间创建新分区。

![image.png](https://ipfs.busy.org/ipfs/QmXd6AxZ1qiZCYKyfMfiCSwEcvecUmfZJfPYrApkbSoaFM)

![image.png](https://ipfs.busy.org/ipfs/QmUbB24xnMEUcuvn1fa5EFcrZ6otH1YqUcwmAJL1fdFtn7)

- 设置新分区挂载到根目录，文件系统选择ext4，分区类型为GPT。

![image.png](https://ipfs.busy.org/ipfs/QmcEqL6K7WoezdnHhCMj75AjdMUrRj5X8aAZ2VKAg479fP)

- 选中Window10 创建的EFI分区，大小是100M，类型是FAT32，点击编辑。

![image.png](https://ipfs.busy.org/ipfs/QmWvcsG8H11YcnQKLiyAaPyNYvHiFKRzUWdRNQAhy5Zk68)

- **分区内容选择保留**，挂载点设置为 /boot/efi，标记选中boot和esp。

![image.png](https://ipfs.busy.org/ipfs/QmdwBbP98oBSjcsA7Ei3xeb3zCpCcySMNrcJk5TgErzzjT)

- 设置用户和密码

![image.png](https://ipfs.busy.org/ipfs/QmTcGWSuYj1H4joGMer2cAaCyDcAtYnmeTWsXavu3av2Jm)

- 后面就可以安装了，等待安装完毕。

![image.png](https://ipfs.busy.org/ipfs/QmP5f2HPtrKKdZ4GNSTvZMBUTGABpVfasoRXYm3Rn6esrq)

![image.png](https://ipfs.busy.org/ipfs/QmeEG8KGFK2G3KgfxivvGAvV1LsKnxRWqf7M8VJmLB5upD)

## 完成安装

- 选择启动项进入相应的系统

![image.png](https://ipfs.busy.org/ipfs/QmUV7UUCXqi4MQ8AL3HYxD5MmhseiXH8wajxB7k6zkHKpd)

- 进入Mamjaro

![image.png](https://ipfs.busy.org/ipfs/QmVp1MEHdn1KJp1t9xzkXkJXymgKpuJGRYnztToP9oFAQC)

- 进入 Windows 10

![image.png](https://ipfs.busy.org/ipfs/QmSfUUzCs7oeqXrSt8cukajwNegdrYRchM1XyGaNvQAKVL)