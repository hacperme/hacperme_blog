---
id: 1373
title: 'jupyter notebook 学习笔记'
date: '2018-06-08T23:59:31+08:00'
author: hacper
excerpt: 'jupyter notebook 学习笔记'
layout: post
guid: 'https://hacperme.com/?p=1373'
permalink: /2018/06/08/jupyter_notebook_learnning_note/
views:
    - '2097'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - ''
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - jupyter
    - notebook
    - python
---

## 快捷键

1. 查看快捷键

在右上角的== help->keyboard Shortcuts ==  
或者在命令模式按 **H**

2. 常用快捷键

**Command Mode (press Esc to enable)**

- <span class="burk">F : find and replace</span>
- Enter : enter edit mode
- Shift-Enter : run cell, select below
- Ctrl-Enter : run selected cells
- Alt-Enter : run cell and insert below
- <span class="girk">Y : change cell to code</span>
- <span class="burk"><span class="mark"><span class="mark"><span class="mark">M : change cell to markdown</span></span></span></span>
- R : change cell to raw
- <span class="burk"><span class="mark">1 : change cell to heading 1</span></span>
- 2 : change cell to heading 2
- K : select cell abov- F : find and replace
- Enter : enter edit mode
- <span class="burk">P : open the command palette</span>
- Shift-Enter : run cell, select below
- Ctrl-Enter : run selected cells
- Alt-Enter : run cell and insert below
- Y : change cell to code
- 1 : change cell to heading 1
- 2 : change cell to heading 2
- K : select cell above
- Up : select cell above
- Down : select cell below
- J : select cell below
- Shift-K : extend selected cells above
- Shift-Up : extend selected cells above
- Shift-Down : extend selected cells below
- Shift-J : extend selected cells below
- A : insert cell above
- B : insert cell below
- X : cut selected cells
- C : copy selected cells
- Z : undo cell deletion
- D,D : delete selected cells
- Shift-M : merge selected cells, or current cell with cell below if only one cell is selected
- L : toggle line numbers
- H : show keyboard shortcuts
- Shift-Space : scroll notebook up
- Space : scroll notebook downe

**Edit Mode (press Enter to enable)**

- <span class="mark">Tab : code completion or indent</span>
- <span class="burk">Shift-Tab: tooltip (e.g. to get function arguments)</span>
- Ctrl-A : select all
- Ctrl-Z : undo
- Ctrl-/ : comment
- Esc : enter command mode
- Shift-Enter : run cell, select below
- Ctrl-Enter : run selected cells
- Alt-Enter : run cell and insert below
- Ctrl-Shift-Minus : split cell at cursor

## 查看文档

- 在 help 菜单有常用库文档的连接，包括 NumPy, Pandas, SciPy 和 Matplotlib.
- 在一个库，方法或变量前加上 ?，你可以获得它的一个快速语法说明

```python
  import random
  ?random.choice()
  # 查看帮助

```

```python
??random.choice()
# 查看源码

```

```bash
Signature: random.choice(seq)

Docstring: Choose a random element from a non-empty sequence.

File: i:\python364\lib\random.py

Type: method

```

## Jupyter Magic Commands

```python
%lsmagic
# 查看所有魔法命令

```

```bat
Available line magics:
%alias  %alias_magic  %autocall  %automagic  %autosave  %bookmark  %cd  %clear  %cls  %colors  %config  %connect_info  %copy  %ddir  %debug  %dhist  %dirs  %doctest_mode  %echo  %ed  %edit  %env  %gui  %hist  %history  %killbgscripts  %ldir  %less  %load  %load_ext  %loadpy  %logoff  %logon  %logstart  %logstate  %logstop  %ls  %lsmagic  %macro  %magic  %matplotlib  %mkdir  %more  %notebook  %page  %pastebin  %pdb  %pdef  %pdoc  %pfile  %pinfo  %pinfo2  %popd  %pprint  %precision  %profile  %prun  %psearch  %psource  %pushd  %pwd  %pycat  %pylab  %qtconsole  %quickref  %recall  %rehashx  %reload_ext  %ren  %rep  %rerun  %reset  %reset_selective  %rmdir  %run  %save  %sc  %set_env  %store  %sx  %system  %tb  %time  %timeit  %unalias  %unload_ext  %who  %who_ls  %whos  %xdel  %xmode

Available cell magics:
%%!  %%HTML  %%SVG  %%bash  %%capture  %%cmd  %%debug  %%file  %%html  %%javascript  %%js  %%latex  %%markdown  %%perl  %%prun  %%pypy  %%python  %%python2  %%python3  %%ruby  %%script  %%sh  %%svg  %%sx  %%system  %%time  %%timeit  %%writefile

Automagic is ON, % prefix IS NOT needed for line magics.

```

- 常用魔法命令

```python
%env AA = '123'
# 设置环境变量

%env
# 列出环境变量

```

```bat
env: AA='123'
{'ACTEL_FOR_ALTIUM_OVERRIDE': ' ',
 'ALLUSERSPROFILE': 'D:\\ProgramData',
 'ALTERA_FOR_ALTIUM_OVERRIDE': ' ',
 'APPDATA': 'D:\\Users\\tracis\\AppData\\Roaming',
 'CLASSPATH': ' .;I:\\Java\\jdk1.8.0_101/lib/dt.jar;I:\\Java\\jdk1.8.0_101/lib/tools.jar',
 'COMMONPROGRAMFILES': 'D:\\Program Files (x86)\\Common Files',
 'COMMONPROGRAMFILES(X86)': 'D:\\Program Files (x86)\\Common Files',
 'COMMONPROGRAMW6432': 'D:\\Program Files\\Common Files',
 'COMPUTERNAME': 'HACPER-PC',
 'COMSPEC': 'D:\\WINDOWS\\system32\\cmd.exe',
 'DRIVERDATA': 'D:\\Windows\\System32\\Drivers\\DriverData',
 'ENVCONTAINERTELEMETRYAPICMDLINE': '-st "D:\\Program Files\\NVIDIA Corporation\\NvContainer\\NvContainerTelemetryApi.dll"',
 'ENVCONTAINERTELEMETRYAPICMDLINEX86': '-st "D:\\Program Files (x86)\\NVIDIA Corporation\\NvContainer\\NvContainerTelemetryApi.dll"',
 'FP_NO_HOST_CHECK': 'NO',
 'HOMEDRIVE': 'D:',
 'HOMEPATH': '\\Users\\tracis',
 'JAVA_HOME': 'I:\\Java\\jdk1.8.0_101',
 'K2PDFOPT_CUSTOM0': 'Last Settings;-o D:\\Users\\tracis\\Desktop\\%b_k2opt;',
 'K2PDFOPT_CUSTOM1': '2-column paper;-mode 2col;',
 'K2PDFOPT_CUSTOM2': 'Trim Margins;-mode fw;',
 'K2PDFOPT_WINPOS': '-8 -8 872 720',
 'KMP_DUPLICATE_LIB_OK': 'TRUE',
 'LOCALAPPDATA': 'D:\\Users\\tracis\\AppData\\Local',
 'LOGONSERVER': '\\\\HACPER-PC',
 'MKL_SERIAL': 'YES',
 'NIEXTCCOMPILERSUPP': 'I:\\National Instruments\\Shared\\ExternalCompilerSupport\\C\\',
 'NO_XILINX_DATA_LICENSE': 'HIDDEN',
 'NUMBER_OF_PROCESSORS': '4',
 'ONEDRIVE': 'D:\\Users\\tracis\\OneDrive',
 'OS': 'Windows_NT',
 'PATH': 'I:\\FFMPEG~1\\bin;D:\\PROGRA~1\\Docker\\Docker\\RESOUR~1\\bin;I:\\PHANTO~1.1-W\\bin;I:\\PYTHON~1\\Scripts;I:\\PYTHON~1;I:\\ANDROI~1\\PLATFO~2;D:\\Windows\\System32\\wbem;I:\\Java\\jdk1.8.0_101/bin;I:\\Java\\jdk1.8.0_101/jre/bin;D:\\Windows\\System32;I:\\gcc\\bin;i:\\quartus\\quartus\\bin;D:\\Windows;I:\\OpenVPN\\bin;I:\\calibre\\;I:\\ANDROI~1\\NDK-BU~1;I:\\Redis\\;I:\\Git\\cmd;I:\\MATLAB\\runtime\\win64;I:\\MATLAB\\bin;D:\\Windows\\System32\\WINDOW~1\\v1.0\\;D:\\WINDOWS\\system32;D:\\WINDOWS;D:\\WINDOWS\\System32\\Wbem;D:\\WINDOWS\\System32\\WindowsPowerShell\\v1.0\\;D:\\WINDOWS\\System32\\OpenSSH\\;D:\\Program Files (x86)\\NVIDIA Corporation\\PhysX\\Common;I:\\Process Lasso\\;D:\\Users\\tracis\\AppData\\Local\\Microsoft\\WindowsApps;i:\\python364\\lib\\site-packages\\pywin32_system32;i:\\python364\\lib\\site-packages\\pywin32_system32',
 'PATHEXT': '.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.PY;.PYW',
 'PROCESSOR_ARCHITECTURE': 'x86',
 'PROCESSOR_ARCHITEW6432': 'AMD64',
 'PROCESSOR_IDENTIFIER': 'Intel64 Family 6 Model 61 Stepping 4, GenuineIntel',
 'PROCESSOR_LEVEL': '6',
 'PROCESSOR_REVISION': '3d04',
 'PROGRAMDATA': 'D:\\ProgramData',
 'PROGRAMFILES': 'D:\\Program Files (x86)',
 'PROGRAMFILES(X86)': 'D:\\Program Files (x86)',
 'PROGRAMW6432': 'D:\\Program Files',
 'PSMODULEPATH': 'D:\\WINDOWS\\system32\\WindowsPowerShell\\v1.0\\Modules\\',
 'PUBLIC': 'D:\\Users\\Public',
 'QUARTUS_ROOTDIR': 'i:\\quartus\\quartus',
 'SESSIONNAME': 'Console',
 'SYSTEMDRIVE': 'D:',
 'SYSTEMROOT': 'D:\\WINDOWS',
 'TEMP': 'K:\\temp',
 'TMP': 'K:\\temp',
 'USERDOMAIN': 'HACPER-PC',
 'USERDOMAIN_ROAMINGPROFILE': 'HACPER-PC',
 'USERNAME': 'tracis',
 'USERPROFILE': 'D:\\Users\\tracis',
 'WINDIR': 'D:\\WINDOWS',
 'WINDOWS_TRACING_FLAGS': '3',
 'XILINX_FOR_ALTIUM_OVERRIDE': ' ',
 'JPY_INTERRUPT_EVENT': '1248',
 'IPY_INTERRUPT_EVENT': '1248',
 'JPY_PARENT_PID': '1212',
 'TERM': 'xterm-color',
 'CLICOLOR': '1',
 'PAGER': 'cat',
 'GIT_PAGER': 'cat',
 'MPLBACKEND': 'module://ipykernel.pylab.backend_inline',
 'AA': "'123'"}

```

- 读写文件

```python
%%file hello.txt
Hello, world
This is thing number 1
This is thing number 2
This is thing number 3

```

```bat
Writing hello.txt

```

```python
%pycat hello.txt

```

- %run 可以从.py 文件执行 Python 代码. 它也可以执行其他的 Jupyter notebook,

注意使用%run 并不等同于导入一个 Python 模块.

- %store 命令可以让你在两个不同的 notebook 间传递变量。
- 不带参数的%who 命令将会列出全局范围内存在的所有变量。如果传入参数，比如 str，将会列出指定类型的所有变量。

```python
%who

```

```bat
random


```

- %%time 将会给出 cell 的代码运行一次所花费的时间。

```python
%%time
import time

for i in range(10):
    time.sleep(0.01)

```

```bat
Wall time: 106 ms

```

- %timeit 使用 Python 的 timeit 模块，它将会执行一个语句 100，000 次(默认情况下)，然后给出运行最快 3 次的平均值。

```python
%timeit
import numpy
%timeit numpy.random.normal(size=10)

```

```bat
13.3 µs ± 624 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)

```

- 使用%%writefile 魔法保存 cell 的内容到一个外部文件。%pycat 则刚好相反，并且会向你展示高亮后的外部文件。
- 使用%prun statement\_name 将会产生一个有序表格来展示在该语句中所调用的每个内部函数调用的次数，每次调用的时间与该函数累计运行的时间。

```python
%prun print('hello')

```

```bat
hello

```

- 加上一个分号可以抑制最后一行函数的输出

```python
%matplotlib inline
from matplotlib import pyplot as plt
import numpy
x = numpy.linspace(0, 1, 1000)**1.5

```

```python
plt.hist(x)
# 有输出

```

```bat
(array([216., 126., 106.,  95.,  87.,  81.,  77.,  73.,  71.,  68.]),
 array([0. , 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1. ]),
 &lt;a list of 10 Patch objects&gt;)

```

![output_31_0.png](https://cdn.steemitimages.com/DQmcqi4735WDF1a9WSXsPSnqjqyWyGmQWGvrbCGcMqTv7GW/output_31_0.png)

```python
plt.hist(x);

```

![output_30_1.png](https://cdn.steemitimages.com/DQmcqi4735WDF1a9WSXsPSnqjqyWyGmQWGvrbCGcMqTv7GW/output_30_1.png)

- 执行 shell 命令

```python
!dir

```

```bat
驱动器 K 中的卷是 DATA
卷的序列号是 537C-4A16

 K:\Workspace\python 的目录

2018/06/08  22:03    &lt;DIR&gt;          .
2018/06/08  22:03    &lt;DIR&gt;          ..
2018/06/08  20:39    &lt;DIR&gt;          .ipynb_checkpoints
2018/06/07  13:18    &lt;DIR&gt;          analyses_friend
2018/03/18  00:21    &lt;DIR&gt;          dataanalysis
2018/02/01  23:21    &lt;DIR&gt;          env
2018/06/07  23:20            10,575 fangzheng.ipynb
2018/03/24  22:47         1,780,465 get-pip.py
2018/03/02  00:44    &lt;DIR&gt;          gongzhonghao
2018/05/27  22:12    &lt;DIR&gt;          husespider
2018/03/17  15:53    &lt;DIR&gt;          jupyteconfig
2018/06/08  22:03            29,438 jupyter notebook.ipynb
2018/02/28  15:15    &lt;DIR&gt;          shuzixiaoyuan
2018/06/01  11:29            14,178 Untitled.ipynb
2018/05/25  07:53    &lt;DIR&gt;          wechatbot
2018/05/23  12:14    &lt;DIR&gt;          xuetangzaixian
2018/05/05  14:14             3,252 用 Python 统计字数.ipynb
               5 个文件      1,837,908 字节
              12 个目录 77,272,072,192 可用字节

```

## 书写 LaTeX

$$ P(A \\mid B) = \\frac{P(B \\mid A) \\, P(A)}{P(B)} $$

## 将多个 kernel 的代码组合到一个 notebook

- %%bash
- %%HTML
- %%python2
- %%python3
- %%ruby
- %%perl

加%%是整个 cell 都用那个 kernel 执行代码  
而%是指在行以 kernel 执行相应代码

## 多光标操作

<span class="burk">按住 Alt 进行点击和拖拽鼠标,再按方向键</span>

## The Jupyter output system

notebook 以 HTML 的方式进行展示，cell 的输出也可以是 HTML，所以事实上你可以返回任何东西：视频/音频/图像。

```python
# packages, modules, imports, namespaces
import numpy as np
from scipy.misc import factorial

# function definition with default arguments


def poisson_pmf(k, mu=1):
    """Poisson PMF for value k with rate mu."""
    return mu**k * np.exp(-mu) / factorial(k)

```

```python
# Jupyter notebook "magic" function
# Sets up "inline" plotting
%matplotlib inline

# Importing the seaborn plotting library and setting defaults
import seaborn as sns
sns.set_context("notebook", font_scale=1.5)

# Variable assignment
n = np.arange(10)  # [0, 1, 2, ..., 0]

# Note that poisson_pmf is vectorized
sns.barplot(n, poisson_pmf(n, 2))

# pass is a do-nothing statement -
# Used here to suppress printing of return value for sns.barplot()
pass

```

```bat
i:\python364\lib\site-packages\ipykernel_launcher.py:8: DeprecationWarning: `factorial` is deprecated!
Importing `factorial` from scipy.misc is deprecated in scipy 1.0.0. Use `scipy.special.factorial` instead.

```

![output_40_1.png](https://cdn.steemitimages.com/DQmNfqP9j2ND1ekVKQWC4UauWbza2BPB9GSQ69Ko9bsKDgj/output_40_1.png)

## 安装扩展插件

```python
!python -m pip install https://github.com/ipython-contrib/jupyter_contrib_nbextensions/tarball/master
!python -m  pip install jupyter_nbextensions_configurator
!jupyter contrib nbextension install
!jupyter nbextensions_configurator enable

```

## 参考文章

- [Notes on using Jupyter](http://people.duke.edu/~ccc14/sta-663-2017/00_Jupyter.html)
- [28 Jupyter Notebook tips, tricks, and shortcuts](https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/)