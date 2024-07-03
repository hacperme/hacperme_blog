---
title: "Ubuntu-20.04 python 2.7 安装 pylzma 失败问题处理"
date: 2024-07-03T02:00:18+08:00
lastmod: 2024-07-03T02:00:18+08:00
author: ["hacper"]
tags:
   - pylzma
   - python
   - Ubuntu
   - linux
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: true # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true

---

fcm362k sdk 打包 ota 固件的时候，需要用到 python 的 pylzma 模块，使用的是 python 2.7。

```bash

❯ pip install pylzma
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Defaulting to user installation because normal site-packages is not writeable
Collecting pylzma
  Downloading pylzma-0.5.0.tar.gz (4.2 MB)
     |████████████████████████████████| 4.2 MB 6.0 MB/s 
Building wheels for collected packages: pylzma
  Building wheel for pylzma (setup.py) ... error
  ERROR: Command errored out with exit status 1:
   command: /usr/bin/python2.7 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-wMwVPQ/pylzma/setup.py'"'"'; __file__='"'"'/tmp/pip-install-wMwVPQ/pylzma/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' bdist_wheel -d /tmp/pip-wheel-2wm2G0
       cwd: /tmp/pip-install-wMwVPQ/pylzma/
  Complete output (27 lines):
  running bdist_wheel
  running build
  running build_py
  creating build
  creating build/lib.linux-x86_64-2.7
  copying py7zlib.py -> build/lib.linux-x86_64-2.7
  running build_ext
  There is a workaround to now inherit optimization CFLAGS when compiling wheels.
  To enable this, set APPLY_LP2002043_UBUNTU_CFLAGS_WORKAROUND in your
  environment. See LP: https://launchpad.net/bugs/2002043 for further context.
  APPLY_LP2002043_UBUNTU_CFLAGS_WORKAROUND not detected.
  /tmp/pip-install-wMwVPQ/pylzma/setup.py:107: UnsupportedPlatformWarning: Multithreading is not supported on the platform "linux2",
  please contact mail@joachim-bauch.de for more informations.
    please contact mail@joachim-bauch.de for more informations.""" % (sys.platform), UnsupportedPlatformWarning)
  building 'pylzma' extension
  creating build/temp.linux-x86_64-2.7
  creating build/temp.linux-x86_64-2.7/src
  creating build/temp.linux-x86_64-2.7/src/pylzma
  creating build/temp.linux-x86_64-2.7/src/sdk
  creating build/temp.linux-x86_64-2.7/src/sdk/C
  creating build/temp.linux-x86_64-2.7/src/compat
  x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-CxOYiX/python2.7-2.7.18=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DPY_SSIZE_T_CLEAN=1 -DWITH_COMPAT=1 -DPYLZMA_VERSION=0.5.0 -D_7ZIP_ST=1 -Isrc/sdk/C -I/usr/include/python2.7 -c src/pylzma/pylzma.c -o build/temp.linux-x86_64-2.7/src/pylzma/pylzma.o
  src/pylzma/pylzma.c:26:10: fatal error: Python.h: No such file or directory
     26 | #include <Python.h>
        |          ^~~~~~~~~~
  compilation terminated.
  error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
  ----------------------------------------
  ERROR: Failed building wheel for pylzma
  Running setup.py clean for pylzma
Failed to build pylzma
Installing collected packages: pylzma
    Running setup.py install for pylzma ... error
    ERROR: Command errored out with exit status 1:
     command: /usr/bin/python2.7 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-wMwVPQ/pylzma/setup.py'"'"'; __file__='"'"'/tmp/pip-install-wMwVPQ/pylzma/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /tmp/pip-record-LvdYpy/install-record.txt --single-version-externally-managed --user --prefix= --compile --install-headers /home/xinqiang/.local/include/python2.7/pylzma
         cwd: /tmp/pip-install-wMwVPQ/pylzma/
    Complete output (27 lines):
    running install
    running build
    running build_py
    creating build
    creating build/lib.linux-x86_64-2.7
    copying py7zlib.py -> build/lib.linux-x86_64-2.7
    running build_ext
    There is a workaround to now inherit optimization CFLAGS when compiling wheels.
    To enable this, set APPLY_LP2002043_UBUNTU_CFLAGS_WORKAROUND in your
    environment. See LP: https://launchpad.net/bugs/2002043 for further context.
    APPLY_LP2002043_UBUNTU_CFLAGS_WORKAROUND not detected.
    /tmp/pip-install-wMwVPQ/pylzma/setup.py:107: UnsupportedPlatformWarning: Multithreading is not supported on the platform "linux2",
    please contact mail@joachim-bauch.de for more informations.
      please contact mail@joachim-bauch.de for more informations.""" % (sys.platform), UnsupportedPlatformWarning)
    building 'pylzma' extension
    creating build/temp.linux-x86_64-2.7
    creating build/temp.linux-x86_64-2.7/src
    creating build/temp.linux-x86_64-2.7/src/pylzma
    creating build/temp.linux-x86_64-2.7/src/sdk
    creating build/temp.linux-x86_64-2.7/src/sdk/C
    creating build/temp.linux-x86_64-2.7/src/compat
    x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-CxOYiX/python2.7-2.7.18=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DPY_SSIZE_T_CLEAN=1 -DWITH_COMPAT=1 -DPYLZMA_VERSION=0.5.0 -D_7ZIP_ST=1 -Isrc/sdk/C -I/usr/include/python2.7 -c src/pylzma/pylzma.c -o build/temp.linux-x86_64-2.7/src/pylzma/pylzma.o
    src/pylzma/pylzma.c:26:10: fatal error: Python.h: No such file or directory
       26 | #include <Python.h>
          |          ^~~~~~~~~~
    compilation terminated.
    error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
    ----------------------------------------
ERROR: Command errored out with exit status 1: /usr/bin/python2.7 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-wMwVPQ/pylzma/setup.py'"'"'; __file__='"'"'/tmp/pip-install-wMwVPQ/pylzma/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /tmp/pip-record-LvdYpy/install-record.txt --single-version-externally-managed --user --prefix= --compile --install-headers /home/xinqiang/.local/include/python2.7/pylzma Check the logs for full command output.


```

一安装就报错，折腾老半天:
```
 x86_64-linux-gnu-gcc -pthread -fno-strict-aliasing -Wdate-time -D_FORTIFY_SOURCE=2 -g -fdebug-prefix-map=/build/python2.7-CxOYiX/python2.7-2.7.18=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -DPY_SSIZE_T_CLEAN=1 -DWITH_COMPAT=1 -DPYLZMA_VERSION=0.5.0 -D_7ZIP_ST=1 -Isrc/sdk/C -I/usr/include/python2.7 -c src/pylzma/pylzma.c -o build/temp.linux-x86_64-2.7/src/pylzma/pylzma.o
  src/pylzma/pylzma.c:26:10: fatal error: Python.h: No such file or directory
     26 | #include <Python.h>
        |          ^~~~~~~~~~
  compilation terminated.
  error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
```

后面才发现是需要安装 libpython2.7-dev 包
```bash
sudo apt-get install libpython2.7-dev
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libpython2.7
The following NEW packages will be installed:
  libpython2.7 libpython2.7-dev
0 upgraded, 2 newly installed, 0 to remove and 19 not upgraded.
Need to get 3506 kB of archives.
After this operation, 17.3 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 libpython2.7 amd64 2.7.18-1~20.04.4 [1038 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 libpython2.7-dev amd64 2.7.18-1~20.04.4 [2468 kB]
Fetched 3506 kB in 3s (1033 kB/s)           
Selecting previously unselected package libpython2.7:amd64.
(Reading database ... 47106 files and directories currently installed.)
Preparing to unpack .../libpython2.7_2.7.18-1~20.04.4_amd64.deb ...
Unpacking libpython2.7:amd64 (2.7.18-1~20.04.4) ...
Selecting previously unselected package libpython2.7-dev:amd64.
Preparing to unpack .../libpython2.7-dev_2.7.18-1~20.04.4_amd64.deb ...
Unpacking libpython2.7-dev:amd64 (2.7.18-1~20.04.4) ...
Setting up libpython2.7:amd64 (2.7.18-1~20.04.4) ...
Setting up libpython2.7-dev:amd64 (2.7.18-1~20.04.4) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.16) ...

fcm362k_open on  main [!+?⇡] via △ v3.28.2 took 32s 

```

再用pip 安装 pylzma，搞定。

```bash
❯ pip install pylzma
DEPRECATION: Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality.
Defaulting to user installation because normal site-packages is not writeable
Collecting pylzma
  Using cached pylzma-0.5.0.tar.gz (4.2 MB)
Building wheels for collected packages: pylzma
  Building wheel for pylzma (setup.py) ... done
  Created wheel for pylzma: filename=pylzma-0.5.0-cp27-cp27mu-linux_x86_64.whl size=141946 sha256=b377270f54367f030a7250e6f01a76e9d05262f124bd73bf039bc82425a1570c
  Stored in directory: /home/xinqiang/.cache/pip/wheels/a3/08/31/80d21f91a9a7bf972721aee8a821c96311b65781dda8a6411a
Successfully built pylzma
Installing collected packages: pylzma
Successfully installed pylzma-0.5.0

```
