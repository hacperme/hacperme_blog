---
id: 1604
title: 'Learn Python with Steem #08 笔记'
date: '2018-08-16T06:41:31+08:00'
author: hacper
excerpt: "## 划重点\n\n- 模块、包\n\n 模块是一个XXX.py文件，Python中以模块的方式组织代码片段（函数，类，变量）。\n \n 包是一个目录，里面有一些模块或者子目录，Python中以包的方式管理模块。\n\n\n- 使用模块\n\n 要使用模块，需要先导入模块，使用关键字import来导入模块。\n \n 这是我们使用Python标准库和第三方库的方式。\n\n\n- 安装模块\n \n 第三方模块需要自己安装，可以通过Python的包管理工具pip安装、还可以下载模块源码来安装。\n"
layout: post
guid: 'https://hacperme.com/?p=1604'
permalink: /2018/08/16/learn-python-with-steem-08-%e7%ac%94%e8%ae%b0/
Steempress_sp_steem_publish:
    - '1'
views:
    - '2609'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - learnpythonwithsteem08-5w4cklx91x
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - da-learnpythonwithsteem
    - python
---

# Learn Python with Steem #08 笔记

- - - - - -

\[toc\]

## 划重点

- 模块、包 模块是一个XXX.py文件，Python中以模块的方式组织代码片段（函数，类，变量）。
  
  包是一个目录，里面有一些模块或者子目录，Python中以包的方式管理模块。
- 使用模块
  
  要使用模块，需要先导入模块，使用关键字import来导入模块。
  
  这是我们使用Python标准库和第三方库的方式。
- 安装模块
  
  第三方模块需要自己安装，可以通过Python的包管理工具pip安装、还可以下载模块源码来安装。

## 编程练习

```python
from steem import Steem
s = Steem()
balance = s.get_account('yjcps')['sbd_balance']
print(balance)

```

```
4.484 SBD

```

```python
%%file check_balance.py

from steem import Steem
import sys

account_name = sys.argv[1]
s = Steem()
balance = s.get_account(account_name)['sbd_balance']
print(balance)

```

```
Overwriting check_balance.py

```

```python
!python check_balance.py yjcps

```

```
4.484 SBD

```

```python
!python check_balance.py dapeng

```

```
129.840 SBD

```

## 补充

### pip的安装与使用

- 安装pip

先下载get-pip.py文件

> curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

也可以在浏览器访问网址 https://bootstrap.pypa.io/get-pip.py ，直接下载保存文件。

然后终端用Python执行这个文件

> python get-pip.py

- 升级pip

```python
!pip install -U pip

```

```
Requirement already up-to-date: pip in i:\python~1\lib\site-packages (18.0)

```

```python
!python -m pip install -U pip

```

```
Requirement already up-to-date: pip in i:\python~1\lib\site-packages (18.0)

```

- 用pip安装Python包

```python
!pip install requests

```

```
Collecting requests
  Downloading https://files.pythonhosted.org/packages/65/47/7e02164a2a3db50ed6d8a6ab1d6d60b69c4c3fdf57a284257925dfc12bda/requests-2.19.1-py2.py3-none-any.whl (91kB)
Requirement already satisfied: chardet<3.1.0,>=3.0.2 in i:\python~1\lib\site-packages (from requests) (3.0.4)
Requirement already satisfied: urllib3<1.24,>=1.21.1 in i:\python~1\lib\site-packages (from requests) (1.22)
Requirement already satisfied: idna<2.8,>=2.5 in i:\python~1\lib\site-packages (from requests) (2.6)
Requirement already satisfied: certifi>=2017.4.17 in i:\python~1\lib\site-packages (from requests) (2018.4.16)
Installing collected packages: requests
Successfully installed requests-2.19.1


wxpy 0.3.9.8 has requirement itchat==1.2.32, but you'll have itchat 1.3.10 which is incompatible.
pyspider 0.3.10 has requirement tornado<=4.5.3,>=3.2, but you'll have tornado 5.0.2 which is incompatible.

```

```python
# 指定包的版本
!pip install requests==2.17.1     

```

```
Collecting requests==2.17.1
  Downloading https://files.pythonhosted.org/packages/50/41/f6fdaf24a80c726a72f76b15869a20734b7a527081129a380ddce99ffae0/requests-2.17.1-py2.py3-none-any.whl (87kB)
Requirement already satisfied: chardet<3.1.0,>=3.0.2 in i:\python~1\lib\site-packages (from requests==2.17.1) (3.0.4)
Requirement already satisfied: urllib3<1.22,>=1.21.1 in i:\python~1\lib\site-packages (from requests==2.17.1) (1.21.1)
Requirement already satisfied: certifi>=2017.4.17 in i:\python~1\lib\site-packages (from requests==2.17.1) (2018.4.16)
Requirement already satisfied: idna<2.6,>=2.5 in i:\python~1\lib\site-packages (from requests==2.17.1) (2.5)
Installing collected packages: requests
Successfully installed requests-2.17.1


wxpy 0.3.9.8 has requirement itchat==1.2.32, but you'll have itchat 1.3.10 which is incompatible.
pyspider 0.3.10 has requirement tornado<=4.5.3,>=3.2, but you'll have tornado 5.0.2 which is incompatible.

```

- 安装Wheels格式的包

可以在 https://pypi.org/ 网站搜索Python包，下载与你环境对应的wheel文件

![图片.png](https://ipfs.busy.org/ipfs/QmY9kzBawm5aN53Yq846Hc2PeX33NSGSi3iYz1KYeKtSBj)

```python
# 安装命令

!pip install K:\scrypt-0.8.5-cp36-cp36m-win32.whl

```

```
Requirement already satisfied: scrypt==0.8.5 from file:///K:/scrypt-0.8.5-cp36-cp36m-win32.whl in i:\python~1\lib\site-packages (0.8.5)


Requirement 'K:\\scrypt-0.8.5-cp36-cp36m-win32.whl' looks like a filename, but the file does not exist

```

- 指定安装的镜像源

一般来说，使用国内的镜像源按Python包速度更快、更顺畅。

- V2EX：http://pypi.v2ex.com/simple
- 豆瓣：http://pypi.douban.com/simple
- 中国科学技术大学：http://pypi.mirrors.ustc.edu.cn/simple/
- 清华：https://pypi.tuna.tsinghua.edu.cn/simple

```python
!pip install -i https://pypi.douban.com/simple/ requests

```

```
Looking in indexes: https://pypi.douban.com/simple/
Collecting requests
  Downloading https://pypi.doubanio.com/packages/65/47/7e02164a2a3db50ed6d8a6ab1d6d60b69c4c3fdf57a284257925dfc12bda/requests-2.19.1-py2.py3-none-any.whl (91kB)
Requirement already satisfied: urllib3<1.24,>=1.21.1 in i:\python~1\lib\site-packages (from requests) (1.21.1)
Requirement already satisfied: certifi>=2017.4.17 in i:\python~1\lib\site-packages (from requests) (2018.4.16)
Requirement already satisfied: idna<2.8,>=2.5 in i:\python~1\lib\site-packages (from requests) (2.5)
Requirement already satisfied: chardet<3.1.0,>=3.0.2 in i:\python~1\lib\site-packages (from requests) (3.0.4)
Installing collected packages: requests
Successfully installed requests-2.19.1


wxpy 0.3.9.8 has requirement itchat==1.2.32, but you'll have itchat 1.3.10 which is incompatible.
pyspider 0.3.10 has requirement tornado<=4.5.3,>=3.2, but you'll have tornado 5.0.2 which is incompatible.

```

```python
# 升级
!pip install --upgrade requests 

```

```
Collecting requests
  Using cached https://files.pythonhosted.org/packages/65/47/7e02164a2a3db50ed6d8a6ab1d6d60b69c4c3fdf57a284257925dfc12bda/requests-2.19.1-py2.py3-none-any.whl
Requirement already satisfied, skipping upgrade: chardet<3.1.0,>=3.0.2 in i:\python~1\lib\site-packages (from requests) (3.0.4)
Requirement already satisfied, skipping upgrade: certifi>=2017.4.17 in i:\python~1\lib\site-packages (from requests) (2018.4.16)
Requirement already satisfied, skipping upgrade: idna<2.8,>=2.5 in i:\python~1\lib\site-packages (from requests) (2.5)
Requirement already satisfied, skipping upgrade: urllib3<1.24,>=1.21.1 in i:\python~1\lib\site-packages (from requests) (1.21.1)
Installing collected packages: requests
  Found existing installation: requests 2.17.1
    Uninstalling requests-2.17.1:
      Successfully uninstalled requests-2.17.1
Successfully installed requests-2.19.1


wxpy 0.3.9.8 has requirement itchat==1.2.32, but you'll have itchat 1.3.10 which is incompatible.
pyspider 0.3.10 has requirement tornado<=4.5.3,>=3.2, but you'll have tornado 5.0.2 which is incompatible.

```

- 用pip卸载Python包

```python
!pip uninstall requests -y

```

```
Uninstalling requests-2.19.1:
  Successfully uninstalled requests-2.19.1

```

- 查看哪些包可升级

```python
!pip list --outdated

```

```
Package               Version   Latest    Type 
--------------------- --------- --------- -----
aiohttp               3.2.1     3.3.2     wheel
astroid               1.6.4     2.0.4     wheel
Automat               0.6.0     0.7.0     wheel
beautifulsoup4        4.6.0     4.6.3     wheel
bitarray              0.8.1     0.8.3     sdist
certifi               2018.4.16 2018.8.13 wheel
...
twisted               17.9.0    18.7.0    sdist
urllib3               1.21.1    1.23      wheel
WeRoBot               1.4.1     1.6.0     wheel
widgetsnbextension    3.2.1     3.4.0     wheel
WsgiDAV               2.3.0     2.4.1     wheel
wxPython              4.0.1     4.0.3     wheel
yarl                  1.2.4     1.2.6     wheel

```

- 查看包的安装信息

```python
 !pip show --files requests

```

```
Name: requests
Version: 2.19.1
Summary: Python HTTP for Humans.
Home-page: http://python-requests.org
Author: Kenneth Reitz
Author-email: me@kennethreitz.org
License: Apache 2.0
Location: i:\python~1\lib\site-packages
Requires: idna, urllib3, chardet, certifi
Required-by: yarg, wxpy, WeRoBot, requests-download, pyspider, pynsist, itchat, httpie
Files:
  requests-2.19.1.dist-info\DESCRIPTION.rst
  requests-2.19.1.dist-info\INSTALLER
  requests-2.19.1.dist-info\LICENSE.txt
  ...
  requests\status_codes.py
  requests\structures.py
  requests\utils.py

```

- 导出包的安装信息到指定文件

```python
!pip freeze > requirements.txt

```

```python
%pycat requirements.txt

```

```
aiohttp==3.2.1
appdirs==1.4.3
asn1crypto==0.24.0
astroid==1.6.4
async-timeout==3.0.0
...
windrose==1.6.3
wrapt==1.10.11
WsgiDAV==2.3.0
wxpy==0.3.9.8
wxPython==4.0.1
xmltodict==0.11.0
yapf==0.22.0
yarg==0.1.9
yarl==1.2.4
zope.interface==4.5.0

```

- 从指定文件安装指定的包

```python
!pip install -r requirements.txt

```

```
Requirement already satisfied: aiohttp==3.2.1 in i:\python~1\lib\site-packages (from -r requirements.txt (line 1)) (3.2.1)
Requirement already satisfied: appdirs==1.4.3 in i:\python~1\lib\site-packages (from -r requirements.txt (line 2)) (1.4.3)
Requirement already satisfied: asn1crypto==0.24.0 in i:\python~1\lib\site-packages (from -r requirements.txt (line 3)) (0.24.0)
Requirement already satisfied: astroid==1.6.4 in d:\users\tracis\appdata\roaming\python\python36\site-packages (from -r requirements.txt (line 4)) (1.6.4)
Requirement already satisfied: async-timeout==3.0.0 in i:\python~1\lib\site-packages (from -r requirements.txt (line 5)) (3.0.0)
...
Requirement already satisfied: pip>=9.0.1 in i:\python~1\lib\site-packages (from pipenv==2018.7.1->-r requirements.txt (line 91)) (18.0)

```

- - - - - -

## \[DA series - Learn Python with Steem\]

- [\[DA series - Learn Python with Steem #01\] 安裝Python、文字編輯器與哈囉！](https://busy.org/@deanliu/da-series-learn-python-with-steem-01-python)
- [\[DA series - Learn Python with Steem #02\] 變數與資料型態](https://busy.org/@deanliu/da-series-learn-python-with-steem-02)
- [\[DA series - Learn Python with Steem #03\] 邏輯判斷](https://busy.org/@deanliu/da-series-learn-python-with-steem-03)
- [\[DA series - Learn Python with Steem #04\] 迴圈](https://busy.org/@deanliu/da-series-learn-python-with-steem-04)
- [\[DA series – Learn Python with Steem #05\] 基本資料結構](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [\[DA series - Learn Python with Steem #06\] 函式](https://busy.org/@deanliu/da-series-learn-python-with-steem-06)
- [\[DA series - Learn Python with Steem #07\] 類別](https://busy.org/@deanliu/da-series-learn-python-with-steem-07)
- [\[DA series - Learn Python with Steem #08\] 函式庫(Modules)的安裝與使用，準備好玩Steem！](https://busy.org/@deanliu/da-series-learn-python-with-steem-08-modules-steem)

## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)
- [Learn Python with Steem #03 笔记](https://busy.org/@yjcps/learnpythonwithsteem03-l5w15fszh9)
- [Learn Python with Steem #04 笔记](https://busy.org/@yjcps/learnpythonwithsteem04-0d8jly8ypt)
- [Learn Python with Steem #05 笔记](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [Learn Python with Steem #06 笔记](https://busy.org/@yjcps/learnpythonwithsteem06-c0tgbg3puu)
- [Learn Python with Steem #07 笔记](https://busy.org/@yjcps/learnpythonwithsteem07-2tx2pvwskh)