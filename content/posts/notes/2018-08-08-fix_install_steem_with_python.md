---
id: 1570
title: 安装steem-python遇到的问题
date: '2018-08-08T07:00:40+08:00'
author: hacper
excerpt: "steem-python是steem官方的Python库，今天在Windows系统上安装这个库的时候遇到了一些问题。\n\n安装steem-python的方法是用pip,在cmd里面输命令就可以了：\n\n> python -m pip install steem\n\n但安装过程不是想象中的那样顺利，其中有两个模块安装不了：ujson和pycrypto\n出错提示:\n> error: Microsoft Visual C++ 14.0 is required. Get it with \"Microsoft Visual C++ Build Tools\": http://landinghub.visualstudio.com/visual-cpp-build-tools"
layout: post
guid: 'https://hacperme.com/?p=1570'
permalink: /2018/08/08/fix_install_steem_with_python/
Steempress_sp_steem_publish:
    - '1'
views:
    - '2303'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - steem-python-2eczkyxbhe
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - pycrypto
    - python
    - steem
    - ujson
    - 'VC 14.0'
---

steem-python是steem官方的Python库，今天在Windows系统上安装这个库的时候遇到了一些问题。

安装steem-python的方法是用pip,在cmd里面输命令就可以了：

> python -m pip install steem

但安装过程不是想象中的那样顺利，其中有两个模块安装不了：ujson和pycrypto  
出错提示:

> error: Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools": http://landinghub.visualstudio.com/visual-cpp-build-tools

既然缺少编译工具，那就按照提示下载安装软件 [Visual Studio 2017 Community](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2017)，其实很不情愿安装这类巨无霸IDE软件，没办法，不装就玩不了。

勾选使用C++的桌面开发，更改安装路径。

![blob.jpg](https://i.loli.net/2018/08/07/5b69734e8ede0.jpg)

安装Visual Studio 2017 Community的时候，发现VC\_redist.x64.exe这个组件安装出错，只安装了Visual Studio这个空壳子，编译工具没装上，反复尝试卸载重装，修复，这个组件就是安装不了。

后来从安装日志发现，这个破玩意要先把旧版本的组件卸载了，再安装新组件，然而在卸载的时候出错了，找不到产品安装源。

这应该是从Windows 7 升级到 Windows 10的遗留问题，自升级到Windows 10后，发现以前的一些软件损坏了，而且旧软件卸载不了，新程序安装不上，自此成为电脑上的毒瘤。解决办法是只能用[MicrosoftProgram\_Install\_and\_Uninstall.meta.diagcab](https://support.microsoft.com/zh-cn/help/17588/fix-problems-that-block-programs-from-being-installed-or-removed)一个一个地清除这些毒瘤，速度贼慢，现在还有一堆损坏的软件没清理。

![blob.jpg](https://i.loli.net/2018/08/07/5b697cdacfafa.jpg)

在清理了microsoft visual c++ 2015 x64 minimum runtime、microsoft visual c++ 2015 x64 additional runtime 等这几个"microsoft visual c++ 2015 XXX"的旧组件之后，再重新安装Visual Studio 2017 Community，一路顺畅，安装过程没有出错，编译工具也装好了。

尝试重新安装steem-python库，ujson安装没问题，但是编译pycrypto的时候还是出错了。

![blob.jpg](https://i.loli.net/2018/08/07/5b69800d67ba0.jpg)

参照网上的解决办法:[python3.6安装pycrypto](https://my.oschina.net/mengyoufengyu/blog/1524422)

- 将I:\\Microsoft\\Microsoft Visual Studio\\2017\\Community\\VC\\Tools\\MSVC\\14.14.26428\\include\\stdint.h文件拷贝到I:\\Windows Kits\\10\\Include\\10.0.17134.0\\ucrt目录下
- 修改I:\\Windows Kits\\10\\Include\\10.0.17134.0\\ucrt\\inttypes.h  
  注释#include <stdint.h>，添加#include "stdint.h"

![blob.jpg](https://i.loli.net/2018/08/07/5b6982c21ff22.jpg)

原来出现这个问题是由于编译器找不到stdint.h头文件，这个解决方法也是简单粗暴，哈哈。

![blob.jpg](https://i.loli.net/2018/08/07/5b69839562f99.jpg)

最后，终于steem-python安装成功啦，花了几个小时在安装问题上。

![blob.jpg](https://i.loli.net/2018/08/07/5b6988745a1eb.jpg)

后来试了试在Windows 10 的 Linux 子系统上安装 steem-python，其中没有出现错误，一次性安装好了。唉，早知道直接在Linux系统下安装得了，又是一番折腾。