---
id: 1268
title: 尝鲜Windows10操作系统遇到的问题
date: '2018-05-02T13:30:05+08:00'
author: hacper
excerpt: "我的笔记本一直在用 Windows7 系统，为了使用Linux，安装了双系统和在Windows上安装了虚拟机，但是双系统只能开机时选择运行其中一个系统，而虚拟机一卡一卡的使用很不顺畅，二者使用起来皆有不便。\n\n看到 win10 现在支持内置Linux子系统（Windows Subsystem for Linux）了，终于有动力升级到 Win10 了，既不用装双系统也不用装虚拟机，现在可以很好地实现 Windows 和 Linux “双开”。"
layout: post
guid: 'https://hacperme.com/?p=1268'
permalink: /2018/05/02/fix_upgrade_to_windows10_error/
views:
    - '1683'
fifu_image_url:
    - 'https://hacperme.com/wp-content/uploads/2018/05/5c5148cc0ef83a26d0ced6b4ce643373.png'
categories:
    - 生活
tags:
    - Windows
---

#### 升级到 Windows 10 系统

我的笔记本一直在用 Windows7 系统，为了使用Linux，安装了双系统和在Windows上安装了虚拟机，但是双系统只能开机时选择运行其中一个系统，而虚拟机一卡一卡的使用很不顺畅，二者使用起来皆有不便。

看到 win10 现在支持内置Linux子系统（Windows Subsystem for Linux）了，终于有动力升级到 Win10 了，既不用装双系统也不用装虚拟机，现在可以很好地实现 Windows 和 Linux “双开”。

在 win10 上安装 Linux 子系统可以参考文章：

- [Windows Subsystem for Linux Documentation](https://docs.microsoft.com/zh-cn/windows/wsl/about "Windows Subsystem for Linux Documentation")
- [在 Windows 10 秋季更新上安装 Linux 子系统](http://edi.wang/post/2017/11/25/install-linux-subsystem-on-windows-10-fcu "在 Windows 10 秋季更新上安装 Linux 子系统")

选择“升级系统”而不是“重装系统”是因为我想保留原 win7 系统的软件及其设置，还有系统的激活信息。如果直接覆盖系统盘安装 win10，有些软件还得重新下载安装，还要想办法激活系统，所以对我来说选择升级会更合适。可以使用官方的媒体创建工具升级到 win10，下载链接：[媒体创建工具](https://www.microsoft.com/zh-cn/software-download/windows10 "媒体创建工具")

实际上用这个工具升级系统贼慢，大半天没得电脑用了。升级之前把原来的 win7 系统备份一下，给自己留一颗后悔药，若是升级失败或者 win10 用不习惯，还可以回到以前的样子。

- - - - - -

#### 死在自动更新上

![](https://hacperme.com/wp-content/uploads/2018/05/d74525220a103101fd84c004a95ceae3.png)  
经过漫长的等待，成功升级到了 win10，看了下以前的软件，个人文档都还在，系统也是已激活的状态，似乎一切都很完美。

新系统还没用多长时间，突然就连不上网，检查发现WiFi没了，在网络适配器里无线网卡是禁用的，而且死活无法启用。在 Windows 的更新记录里看到无线网卡的驱动更新失败，总算找到原因了，自动更新似乎还不能关闭。果然在设备管理器看到是驱动出了问题，更新驱动依然无效。

然后在华硕的官网上下载安装这个笔记本的无线驱动，手痒的我顺带把有线网卡的驱动也下载安装了一下，结果不仅无线网卡没救活，有线网卡也死了。我想，开玩笑吧，官方驱动居然没用！

秉着 Windows “小事重启，大事重装”的使用原则，重新安装了 win10。安装完成之后啥事不干，安安静静地等待自动更新完成，以免再出什么幺蛾子，我算是怕了它了。

- - - - - -

#### 遇到程序无法安装和卸载问题

使用一段时间发现升级 win10 的时候并没有完美的保留原来的软件，一些通过 ".msi" 后缀的安装包安装（Windows Installer）的软件以及服务是没法正常使用的，可能损坏了注册表项。很恼火的是这些损坏了的软件会像毒瘤一样留在电脑上，这些软件无法卸载，下载安装包重新安装也会出错，查看日志发现它要先卸载旧的软件，然后再安装，所以也无法重新安装。真是进退两难。

![](https://hacperme.com/wp-content/uploads/2018/04/5cd2a610d406600c0d77af130a7c68f5.png)

![](https://hacperme.com/wp-content/uploads/2018/04/b83fce008eb0e71317b6f9f29c562762.png)

怎么办？  
在 Microsoft 支持找到了答案，显然我不是第一个遇到这个问题的人。下载运行“修复程序无法安装或卸载的问题”，它不能批量地修复所有损坏的软件，只能一个一个地来，还很慢！

- [如何解决 Windows Installer 错误](https://support.microsoft.com/zh-cn/help/2438651/how-to-troubleshoot-windows-installer-errors "如何解决 Windows Installer 错误")
- [修复阻止程序安装或删除的问题](https://support.microsoft.com/zh-cn/help/17588/fix-problems-that-block-programs-from-being-installed-or-removed "修复阻止程序安装或删除的问题")

- - - - - -

升级 win10 的过程不太顺利，想保留原有的软件和激活信息，以为升级是比较合适的方式，但结果并没有和自己预期的一样，在升级过程中和升级之后还是花了很多时间，这是我尝鲜的代价。