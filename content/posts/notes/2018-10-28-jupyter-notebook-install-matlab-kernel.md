---
id: 1770
title: '如何让 jupyter notebook 支持 MATLAB 程序？'
date: '2018-10-28T06:36:49+08:00'
author: hacper
excerpt: "我之前安装的 MATLAB 的版本是 2017a，曾尝试过安装 jupyter 的 MATLAB 内核，但是失败了，网上说要 2017b 的版本才能用。\n\n正好今天帮别人安装了 MATLAB 2018a，然后顺带在自己的电脑上也安装了新版本，尝试再次安装 jupyter 的内核，结果是成功的。以后可以在 jupyter notebook 上写 MATLAB 程序的笔记了。"
layout: post
guid: 'https://hacperme.com/?p=1770'
permalink: /2018/10/28/jupyter-notebook-install-matlab-kernel/
views:
    - '4746'
classic-editor-remember:
    - classic-editor
categories:
    - 笔记
tags:
    - engines
    - jupyter
    - MATLAB
    - matlab_kernel
    - notebook
    - python
---

我之前安装的 MATLAB 的版本是 2017a，曾尝试过安装 jupyter 的 MATLAB 内核，但是失败了，网上说要 2017b 的版本才能用。

正好今天帮别人安装了 MATLAB 2018a，然后顺带在自己的电脑上也安装了新版本，尝试再次安装 jupyter 的内核，结果是成功的。以后可以在 jupyter notebook 上写 MATLAB 程序的笔记了。

安装过程不复杂，就两部分：

- 安装 The MATLAB Engine API for Python
- 安装 Matlab kernel for Jupyter

## 安装 MATLAB 的 Python 引擎

进入MATLAB的安装目录，找到 extern\\engines\\python这个路径，并拷贝下来。比如我的安装位置是：

`I:\matlab2018a\extern\engines\python`

然后打开 cmd 切换到 I:\\matlab2018a\\extern\\engines\\python 目录下：

```bash
D:\Users\tracis>i:

I:\Listary>cd I:\matlab2018a\extern\engines\python


```

输入安装命令：python setup.py install

```bash
I:\matlab2018a\extern\engines\python>python setup.py install
running install
running build
running build_py
running install_lib
copying build\lib\matlab\engine\_arch.txt -> I:\python36\Lib\site-packages\matlab\engine
running install_egg_info
Removing I:\python36\Lib\site-packages\matlabengineforpython-R2018a-py3.6.egg-info
Writing I:\python36\Lib\site-packages\matlabengineforpython-R2018a-py3.6.egg-info

```

## 安装 jupyter 的 MATLAB 内核

在 cmd 输入安装命令：

`pip install matlab_kernel`

查看已安装的 jupyter 内核：

`jupyter kernelspec list`

```bash
Available kernels:
  matlab     i:\python36\share\jupyter\kernels\matlab
  python3    i:\python36\share\jupyter\kernels\python

```

Available kernels 下有 matlab，说明安装成功了。

## 尝试 MATLAB notebook

启动 jupyter notebook，尝试在 jupyter notebook 中使用写MATLAB程序：

`jupyter notebook`

在 new 菜单下，选择创建 MATLAB 笔记本，以下是测试效果：

![blob.jpg](https://i.loli.net/2018/10/27/5bd418441f7b6.jpg)

创建一个全 1 方阵：  
![blob.jpg](https://i.loli.net/2018/10/27/5bd41b63ae37d.jpg)

画图：  
![blob.jpg](https://i.loli.net/2018/10/27/5bd41b7da84ac.jpg)

- - - - - -

参考资料：

- https://www.mathworks.com/help/matlab/matlab\_external/install-the-matlab-engine-for-python.html
- https://github.com/Calysto/matlab\_kernel