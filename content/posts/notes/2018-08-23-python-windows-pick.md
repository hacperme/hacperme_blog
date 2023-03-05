---
id: 1625
title: 'Python 在 Windows 系统中导入 pick 模块问题'
date: '2018-08-23T00:18:34+08:00'
author: hacper
excerpt: "在Windows上，用Python导入pick模块会出现以下错误：\n\n> ModuleNotFoundError: No module named '_curses'"
layout: post
guid: 'https://hacperme.com/?p=1625'
permalink: /2018/08/23/python-windows-pick/
Steempress_sp_steem_publish:
    - '0'
views:
    - '2201'
classic-editor-remember:
    - classic-editor
categories:
    - 笔记
tags:
    - curses
    - pick
    - python
---

在Windows上，用Python导入pick模块会出现以下错误：

> ModuleNotFoundError: No module named '\_curses'

```python
>>> from pick import pick
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "I:\PYTHON~1\lib\site-packages\pick\__init__.py", line 3, in <module>
    import curses
  File "I:\PYTHON~1\lib\curses\__init__.py", line 13, in <module>
    from _curses import *
ModuleNotFoundError: No module named '_curses'
>>> from pick import pick
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "I:\PYTHON~1\lib\site-packages\pick\__init__.py", line 3, in <module>
    import curses
  File "I:\PYTHON~1\lib\curses\__init__.py", line 13, in <module>
    from _curses import *
ModuleNotFoundError: No module named '_curses'

```

原因就是 curses 这个模块不支持Windows：

```shell
D:\Users\tracis>pip install curses
Collecting curses
  Could not find a version that satisfies the requirement curses (from versions: )
No matching distribution found for curses

```

在pick的安装描述中有一个提醒，说到了这个问题，但是自己开始的时候没注意到。

> Note for Windows: curses wheels can be obtained from http://www.lfd.uci.edu/~gohlke/pythonlibs/#curses, then install it with pip, for example: `pip install curses-2.2-cp27-none-win_amd64.whl`

在 <https://www.lfd.uci.edu/~gohlke/pythonlibs/#curses> 网站下载第三方的wheels 安装包，用 pip 安装即可。

![blob.jpg](https://i.loli.net/2018/08/23/5b7d8c327c03d.jpg)