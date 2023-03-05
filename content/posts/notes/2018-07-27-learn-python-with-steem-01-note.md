---
id: 1528
title: 'Learn Python with Steem #01 笔记'
date: '2018-07-27T00:23:13+08:00'
author: hacper
excerpt: "## 划重点\n\n- 安装Python\n\n 在[官网](https://www.python.org/downloads/)下载Python 3.6，由于Python 3.7是最近推出的，可能一些第三方的库还没跟上它的脚步，还不支持Python3.7。\n\n 安装的时候要添加到PATH环境变量，这样不管你在哪里使用Pytho电脑都找到得到Python啦。\n\n- 安装Sublime\n\n Sublime 不仅仅可以用来编程序，还可以编辑其他文本文件，是把利器。\n\n\n- 编写第一个程序\n\n 万事开头难，但Python的开始学习曲线较平缓，开头不难，要坚持。"
layout: post
guid: 'https://hacperme.com/?p=1528'
permalink: /2018/07/27/learn-python-with-steem-01-note/
views:
    - '942'
Steempress_sp_steem_publish:
    - '0'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - learn-python-with-steem-01
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - 'jupyter noteebook'
    - python
---

## \# Learn Python with Steem #01 笔记

\[toc\]

## 划重点

- 安装Python 在[官网](https://www.python.org/downloads/)下载Python 3.6，由于Python 3.7是最近推出的，可能一些第三方的库还没跟上它的脚步，还不支持Python3.7。
  
  安装的时候要添加到PATH环境变量，这样不管你在哪里使用Pytho电脑都找到得到Python啦。
- 安装Sublime
  
  Sublime 不仅仅可以用来编程序，还可以编辑其他文本文件，是把利器。
- 编写第一个程序
  
  万事开头难，但Python的开始学习曲线较平缓，开头不难，要坚持。

## 编程练习

```python
# 语言是用来交流的工具，用Python打个招呼。


print('Hello Steemians with Python!')

```

```
Hello Steemians with Python!

```

```python
print('Follow @deanliu & @antonsteemit, and learn Python with Steem!')

```

```
Follow @deanliu & @antonsteemit, and learn Python with Steem!

```

```python
# 看看神奇的Python之禅

import this

```

```
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!

```

## 补充（笔记本）

好高兴，今天的Python课程学完了。  
但要是课后的玩耍的时候玩过头了，把今天学的知识忘了怎么办？  
嗯，好学生都有会记笔记啦，忘了就翻翻笔记。  
我准备了个笔记本，叫 jupyter notebook，就用它记录每天学到的Python知识。

看看这个笔记本长什么样吧：

- 运行在浏览器里，和我们打开的网页一样：  
  ![图片.png](https://ipfs.busy.org/ipfs/QmS9CCyt6BtKSNX2bDwgBmc5Ag4wQTSknsbf5zEbP7QpNP)

可以用markdown语法记笔记，还可以直接在笔记本上运行Python程序  
![图片.png](https://ipfs.busy.org/ipfs/QmaoCi9h6cTNqvgFCaNz9fakU9QHUZcgYFUThSHKobHRaG)

![图片.png](https://ipfs.busy.org/ipfs/QmSA6MPr7yyoqD4DvrFgEVSyo9i9Yc1kfMaZn1yxVppnaL)

那如何安装这个笔记本呢？

步骤如下：

- 打开Windows下的命令提示符程序cmd：

![图片.png](https://ipfs.busy.org/ipfs/QmTmvxE2EVMrP9DT8YDTB2M91dtTi3bt19qUAPSwArjFYP)

- 输入下面的命令后，回车：

> python -m pip install jupyter

如果出错，尝试这条命令

> pip install jupyter

![图片.png](https://ipfs.busy.org/ipfs/Qmcn6emMCeCwKJ5jSETGTXaL4YmXtvWEqFngoaiRf1Frxy)

这样就安装好了。

那如何使用这个笔记本呢？

- 在命令提示符程序cmd里输入命令:

> jupyter notebook

然后会生成这么个网址，一般会自动在浏览器里打开，如果没有，可以自己手动复制粘贴到浏览器里打开

![图片.png](https://ipfs.busy.org/ipfs/QmNVTM16YQcLxiraCLDbsVjKmnSnXcVeTYTVyMfAji8phh)

- 新建笔记本

在右上角new菜单选择Python 3：

![图片.png](https://ipfs.busy.org/ipfs/QmNxnkRWALB9tcJk6mU6BmyyEKKd9Wss26TQfVZvE7gv89)

打开的笔记本：  
![图片.png](https://ipfs.busy.org/ipfs/QmdpEAmeonmFgSpSiwq1a5qVMekGNsnYVp6W4f58diCmz5)

- 笔记示例：

输入markdown  
![图片.png](https://ipfs.busy.org/ipfs/QmeSQbGuoWM9UCJZAh8KMk7jjZbDSRJ5maoMLNxsRcyDiA)  
按Shift + Enter（回车）或者 Ctrl + Enter（回车）执行

![图片.png](https://ipfs.busy.org/ipfs/QmZoBEbnXRa9KYasS4DwEricfh4Sd7JXrsF6QD1rY4Rvpp)

输入Python代码

![图片.png](https://ipfs.busy.org/ipfs/Qmcu6wFbJgo2d7rv28JoKgb3r1CZHsEeJLdrwgoYWfd1pq)

按Shift + Enter（回车）或者 Ctrl + Enter（回车）执行  
![图片.png](https://ipfs.busy.org/ipfs/Qmek68UwCYBiMzHGTFs6cfCTpdpAr2qxJ7zN9d61845itz)

- 导出笔记

![图片.png](https://ipfs.busy.org/ipfs/QmcjoDY4ePQv2LemSVb93BezQxF8UD1JyaYBLQiWGf6Abj)

可以导出多种格式。

- - - - - -

这篇文章是在这个笔记本里写的，导出为markdown文件，复制到steemit上发表，感觉如何？赶紧把Python玩起来吧！