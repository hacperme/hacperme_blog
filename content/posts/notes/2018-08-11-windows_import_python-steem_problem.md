---
id: 1585
title: 在Windows导入python-steem遇到的问题
date: '2018-08-11T14:00:53+08:00'
author: hacper
excerpt: 我以为成功安装python-steem库就可以愉快地玩耍了，这个想法还是太单纯，谁知道在导入steem模块的时候又出现问题了：无法导入winrandom模块。
layout: post
guid: 'https://hacperme.com/?p=1585'
permalink: /2018/08/11/windows_import_python-steem_problem/
Steempress_sp_steem_publish:
    - '1'
views:
    - '1636'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - windowspython-steem-vzofcch5pk
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - cn
    - da-learnpythonwithsteem
    - python
    - steem
---

我以为成功安装python-steem库就可以愉快地玩耍了，这个想法还是太单纯，谁知道在导入steem模块的时候又出现问题了：无法导入winrandom模块。

```python
from steem import Steem
s = Steem()
balance = s.get_account('yjcps')['sbd_balance']
print(balance)

```

```
i:\python364\lib\site-packages\Crypto\Random\OSRNG\nt.py in <module>()
     26 __all__ = ['WindowsRNG']
     27 
---> 28 import winrandom
     29 from .rng_base import BaseRNG
     30 

ModuleNotFoundError: No module named 'winrandom'

```

很自然地想到winrandom这个模块是不是没安装啊？拿出pip安装winrandom试试。

> python -m pip install winrandom

在编译winrandom时又有新问题：ValueError: Unknown MS Compiler version 1900

```
 ----------------------------------------
  Failed building wheel for winrandom
  Running setup.py clean for winrandom
Failed to build winrandom
Installing collected packages: winrandom
  Running setup.py install for winrandom ... error
    Complete output from command I:\PYTHON~1\python.exe -u -c "import setuptools, tokenize;__file__='K:\\temp\\pip-install-f7pz4ekx\\winrandom\\setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record K:\temp\pip-record-xtfsjesk\install-record.txt --single-version-externally-managed --compile:
    running install
    running build
    running build_ext
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "K:\temp\pip-install-f7pz4ekx\winrandom\setup.py", line 43, in <module>
        ext_modules=[winrandom1])
     ...
        self.dll_libraries = get_msvcr()
      File "I:\PYTHON~1\lib\distutils\cygwinccompiler.py", line 95, in get_msvcr
        raise ValueError("Unknown MS Compiler version %s " % msc_ver)
    ValueError: Unknown MS Compiler version 1900

    ----------------------------------------

```

网上查到解决办法，修改I:\\PYTHON~1\\lib\\distutils\\cygwinccompiler.py文件，打补丁。

```python
def get_msvcr():
    """Include the appropriate MSVC runtime library if Python was built
    with MSVC 7.0 or later.
    """
    msc_pos = sys.version.find('MSC v.')
    if msc_pos != -1:
        msc_ver = sys.version[msc_pos+6:msc_pos+10]
        if msc_ver == '1300':
            # MSVC 7.0
            return ['msvcr70']
        ...
        elif msc_ver == '1600':
            # VS2010 / MSVC 10.0
            return ['msvcr100']
         ### PATCH###############################
        # INCLUDES NEWEST mscvcr VERSION
        #########################################
        elif msc_ver == '1900':
           # Visual Studio 2015 / Visual C++ 14.0
           # "msvcr140.dll no longer exists" http://blogs.msdn.com/b/vcblog/archive/2014/06/03/visual-studio-14-ctp.aspx
           return ['vcruntime140']
        #########################################

        else:
            raise ValueError("Unknown MS Compiler version %s " % msc_ver)

```

再次尝试安装编译winrandom，还是失败：No such file or directory  
这是找不到src/winrandom.c源文件吗？继续鼓捣鼓捣，感觉自己瞎折腾了一番，没解决问题，玩累了，再次到网上求助。

```
 ----------------------------------------
  Failed building wheel for winrandom
  Running setup.py clean for winrandom
Failed to build winrandom
Installing collected packages: winrandom
  Running setup.py install for winrandom ... error
    Complete output from command I:\PYTHON~1\python.exe -u -c "import setuptools, tokenize;__file__='K:\\temp\\pip-install-g_5exyuk\\winrandom\\setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record K:\temp\pip-record-qp70g1vo\install-record.txt --single-version-externally-managed --compile:
    running install
    running build
    running build_ext
    building 'winrandom' extension
    creating build
    creating build\temp.win32-3.6
    creating build\temp.win32-3.6\Release
    creating build\temp.win32-3.6\Release\src
    I:\gcc\bin\gcc.exe -mdll -O -Wall -II:\PYTHON~1\include -II:\PYTHON~1\include -c src/winrandom.c -o build\temp.win32-3.6\Release\src\winrandom.o
    gcc: error: CreateProcess: No such file or directory
    error: command 'I:\\gcc\\bin\\gcc.exe' failed with exit status 1

    ----------------------------------------

```

网上别人给的解决办法：修改i:\\python364\\lib\\site-packages\\Crypto\\Random\\OSRNG\\nt.py文件，将import winrandom 改成 from . import winrandom

方法有效。原来不是winrandom模块没有安装，而是Python的导入机制找不到模块的路径。先入为主地把思考问题的方向弄错了，花多少时间都是白费。Python的模块导入机制是如何搜索模块路径的？得花时间去了解一下。

```python
# import winrandom
from . import winrandom

```

尝试导入steem模块，又有新问题：OSError: \[WinError 193\] %1 不是有效的 Win32 应用程序。

```
i:\python364\lib\site-packages\scrypt\scrypt.py in <module>()
     13 __version__ = '0.8.6'
     14 
---> 15 _scrypt = cdll.LoadLibrary(imp.find_module('_scrypt')[1])
     16 
     17 _scryptenc_buf = _scrypt.exp_scryptenc_buf

i:\python364\lib\ctypes\__init__.py in LoadLibrary(self, name)
    424 
    425     def LoadLibrary(self, name):
--> 426         return self._dlltype(name)
    427 
    428 cdll = LibraryLoader(CDLL)

i:\python364\lib\ctypes\__init__.py in __init__(self, name, mode, handle, use_errno, use_last_error)
    346 
    347         if handle is None:
--> 348             self._handle = _dlopen(self._name, mode)
    349         else:
    350             self._handle = handle

OSError: [WinError 193] %1 不是有效的 Win32 应用程序。

```

问题出现在加载"\_scrypt"这个模块上，这个模块不是一个.py文件，而是一个编译好的动态库："\_scrypt.cp36-win32.pyd"。  
为什么出现这个问题？  
难道是混用了 32-bit 和 64-bit的程序？通过pip安装的，应该不会弄错版本，而且安装的时候会提示：**XXX is not a supported wheel on this platform.**

难道是"\_scrypt.cp36-win32.pyd"这个程序损坏了？将scrypt模块卸载重装，问题依旧。没招了，不知道该怎么鼓捣了。

到了第二天，突然想了想，当前scrypt模块安装的是最新版：scrypt-0.8.6，要不装旧版本的试试，然后安装了scrypt-0.8.5。

```
D:\Users\tracis>python -m pip uninstall scrypt
Uninstalling scrypt-0.8.6:
  Would remove:
    i:\python~1\lib\site-packages\_scrypt.cp36-win32.pyd
    i:\python~1\lib\site-packages\scrypt-0.8.6.dist-info\*
    i:\python~1\lib\site-packages\scrypt\*
Proceed (y/n)? y
  Successfully uninstalled scrypt-0.8.6

D:\Users\tracis>python -m pip install K:\scrypt-0.8.5-cp36-cp36m-win32.whl
Processing k:\scrypt-0.8.5-cp36-cp36m-win32.whl
Installing collected packages: scrypt
Successfully installed scrypt-0.8.5

```

![blob.jpg](https://i.loli.net/2018/08/11/5b6e68f3747a2.jpg)

导入steem模块成功，运气真好。迷迷糊糊地从坑里爬出来了=\_=。