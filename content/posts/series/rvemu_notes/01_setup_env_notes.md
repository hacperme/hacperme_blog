---
title: "rvemu 学习笔记-搭建开发环境"
date: 2023-05-21T18:09:04+08:00
lastmod: 2023-05-21T18:09:04+08:00
author: ["hacper"]
tags:
    - rvemu
    - docker
    - makefile
    - ubuntu
    - linux
    - risc-v
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "搭建开发环境和总结makefile基础知识" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
---

## 视频

{{< bilibili BV1uY4y1D7bJ >}}

## 搭建环境

计划使用docker 跑 ubuntu 20.04 系统，并将外部工程目录 D:\workspace\rvemu_env 挂载到容器内，方便共享文件。

```powershell
PS C:\Users\hacper> docker run -itd -v D:\workspace\rvemu_env:/rvemu_env  --name rvemu ubuntu:20.04
79a6aa79f8523800b37f878e29a23b6b168808d4588b639a9d24e89743e7c3c3
PS C:\Users\hacper> docker attach 79a6aa79f8523
root@79a6aa79f852:/# ls
bin   dev  home  lib32  libx32  mnt  proc  run        sbin  sys  usr
boot  etc  lib   lib64  media   opt  root  rvemu_env  srv   tmp  var
```

安装开发需要的软件包：clang make  gcc

```bash
root@79a6aa79f852:/# apt update && apt install clang make vim  gcc -y     

root@79a6aa79f852:/# clang -v
clang version 10.0.0-4ubuntu1
Target: x86_64-pc-linux-gnu
Thread model: posix
InstalledDir: /usr/bin
Found candidate GCC installation: /usr/bin/../lib/gcc/x86_64-linux-gnu/9
Found candidate GCC installation: /usr/lib/gcc/x86_64-linux-gnu/9
Selected GCC installation: /usr/bin/../lib/gcc/x86_64-linux-gnu/9
Candidate multilib: .;@m64
Selected multilib: .;@m64
root@79a6aa79f852:/# make -v
GNU Make 4.2.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

root@79a6aa79f852:/rvemu_env/rvemu# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:hsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.4.0-1ubuntu1~20.04.1' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-9-Av3uEd/gcc-9-9.4.0/debian/tmp-nvptx/usr,hsa --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04.1) 
```

再安装  RV64 Newlib 版本的工具链: [riscv64-elf-ubuntu-20.04-nightly-2023.05.19-nightly.tar.gz](https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2023.05.19/riscv64-elf-ubuntu-22.04-nightly-2023.05.19-nightly.tar.gz)。下载到共享目录和解压，并添加路径到环境变量。

```bash
root@79a6aa79f852:~# cp riscv64-elf-ubuntu-20.04-nightly-2023.05.19-nightly.tar.gz ~
root@79a6aa79f852:~# tar -xvf riscv64-elf-ubuntu-20.04-nightly-2023.05.19-nightly.tar.gz
root@79a6aa79f852:~# echo "export PATH=$PATH:/root/riscv/bin" > rvemu_env.sh
root@79a6aa79f852:~# source rvemu_env.sh

root@79a6aa79f852:~# riscv64-unknown-elf-gcc -v
Using built-in specs.
COLLECT_GCC=riscv64-unknown-elf-gcc
COLLECT_LTO_WRAPPER=/root/riscv/bin/../libexec/gcc/riscv64-unknown-elf/12.2.0/lto-wrapper
Target: riscv64-unknown-elf
Configured with: /home/runner/work/riscv-gnu-toolchain/riscv-gnu-toolchain/gcc/configure --target=riscv64-unknown-elf --prefix=/opt/riscv --disable-shared --disable-threads --enable-languages=c,c++ --with-pkgversion=g2ee5e430018 --with-system-zlib --enable-tls --with-newlib --with-sysroot=/opt/riscv/riscv64-unknown-elf --with-native-system-header-dir=/include --disable-libmudflap --disable-libssp --disable-libquadmath --disable-libgomp --disable-nls --disable-tm-clone-registry --src=.././gcc --disable-multilib --with-abi=lp64d --with-arch=rv64gc --with-tune=rocket --with-isa-spec=20191213 'CFLAGS_FOR_TARGET=-Os    -mcmodel=medlow' 'CXXFLAGS_FOR_TARGET=-Os    -mcmodel=medlow'
Thread model: single
Supported LTO compression algorithms: zlib
gcc version 12.2.0 (g2ee5e430018)

```

不安装 gcc 使用 riscv64-unknown-elf-gcc 编译代码会出现找不到  libmpc.so.3 的错误，所以在上一步把 gcc 也安装上。

```bash
root@79a6aa79f852:/rvemu_env/rvemu# riscv64-unknown-elf-gcc test.c -o test
/root/riscv/bin/../libexec/gcc/riscv64-unknown-elf/12.2.0/cc1: error while loading shared libraries: libmpc.so.3: cannot open shared object file: No such file or directory
```

## vscode 连接到 docker 容器

vscode 可以直接连接到 docker 容器，不需要再配置网络、sshd 这些，也挺方便。在远程连接界面选择 Attach to Running Contaner 即可。编译跑一下已有的完整代码，验证环境是正常的。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image.1lg5k9upb3c0.png)

## makefile 基础知识

借助第一课的基础 makefile 文件，对 makefile 的基础知识做一个整理。

```makefile
CFLAGS=-O3 -Wall -Werror -Wimplicit-fallthrough
SRCS=$(wildcard src/*.c)
HDRS=$(wildcard src/*.h)
OBJS=$(patsubst src/%.c, obj/%.o, $(SRCS))
CC=clang

rvemu: $(OBJS)
	$(CC) $(CFLAGS) -lm -o $@ $^ $(LDFLAGS)

$(OBJS): obj/%.o: src/%.c $(HDRS)
	@mkdir -p $$(dirname $@)
	$(CC) $(CFLAGS) -c -o $@ $<

clean:
	rm -rf rvemu obj/

.PHONY: clean

```

### makefile 的编写规则

makefile 文件记录了一个工程怎样构建编译得到目标产物的规则，makefile 的规则主要由目标、依赖和命令三部分组成。简单地理解 makefile 规则：想要生成的目标文件依赖于一个或者多个文件，目标的生成规则定义在命令中，一旦依赖文件有更新，就会执行命令构建目标文件。

makefile 通过文件的时间戳来判断是否文件有更新。



makefile文件规则：

```
目标：依赖
<TAB>命令
```

目标：可以是可执行文件、中间文件、或者标签（伪目标）。

依赖：可以是源文件、其他目标。

命令：任何 shell 命令

如：

```makefile
rvemu: $(OBJS)
	$(CC) $(CFLAGS) -lm -o $@ $^ $(LDFLAGS)
```

伪目标不会生成目标文件，只执行命令，用于执行某项特定的任务。

如：

```makefile
clean:
	rm -rf rvemu obj/

.PHONY: clean
```

### makefile 变量

makefile 里面可以定义和使用变量，可分为预定义变量、自定义变量和自动变量三部分。

#### 常见的预定义变量

| AR       | 归档维护程序的程序名，默认值为ar  |
| -------- | --------------------------------- |
| ARFLAGS  | 归档维护程序的选项                |
| AS       | 汇编程序的名称，默认值为as        |
| ASFLAGS  | 汇编程序的选项                    |
| CC       | C编译器的名称，默认值为cc         |
| CFLAGS   | C编译器的选项                     |
| CPP      | C预编译器的名称，默认值为$(CC) -E |
| CPPFLAGS | C预编译的选项                     |
| CXX      | C++编译器的名称，默认值为g++      |
| CXXFLAGS | C++编译器的选项                   |

环境变量也可以引用，也看作是预定义变量。

如：

```makefile
CFLAGS=-O3 -Wall -Werror -Wimplicit-fallthrough
CC=clang
```

#### 自定义变量和使用

变量名可以数字开头，区分大小写，定义方法:

变量名=变量值

如：

```makefile
SRCS=$(wildcard src/*.c)
HDRS=$(wildcard src/*.h)
OBJS=$(patsubst src/%.c, obj/%.o, $(SRCS))
```

使用变量的值：

\$(变量名)或\${变量名}

如：

```makefile
rvemu: $(OBJS)
	$(CC) $(CFLAGS) -lm -o $@ $^ $(LDFLAGS)
```

#### 自动变量

| $@   | 目标名                                                       |
| ---- | ------------------------------------------------------------ |
| $<   | 依赖文件列表中的第一个文件                                   |
| $^   | 依赖文件列表中除去重复文件的部分                             |
| $*   | 不包含扩展名的目标文件名称                                   |
| $+   | 所有的依赖文件，以空格分开，并以出现的先后为序，可能包含重复的依赖文件 |
| $?   | 所有时间戳比目标文件晚的依赖文件，并以空格分开               |
| $%   | 如果目标是归档成员，则该变量表示目标的归档成员名称           |

如：

```makefile
rvemu: $(OBJS)
	$(CC) $(CFLAGS) -lm -o $@ $^ $(LDFLAGS)

$(OBJS): obj/%.o: src/%.c $(HDRS)
	@mkdir -p $$(dirname $@)
	$(CC) $(CFLAGS) -c -o $@ $<
```




### 通配符

在规则中可以使用通配符：

| *    | 匹配任意长度的任意字符序列。例如，`*.c` 匹配所有以 `.c` 结尾的文件。 |
| ---- | ------------------------------------------------------------ |
| ?    | 匹配任意单个字符。例如，`file?.txt` 可以匹配 `file1.txt`, `file2.txt`,等。 |
| []   | 匹配方括号内的任意一个字符。例如，`file[123].txt` 可以匹配 `file1.txt`, `file2.txt`, `file3.txt`。 |

如：

```makefile
SRCS=$(wildcard src/*.c)
clean:
    rm -f *.o
```

在 Makefile 中，`%` 通配符用于模式规则（pattern rules）中，表示匹配任意字符序列。

如：

```makefile
OBJS=$(patsubst src/%.c, obj/%.o, $(SRCS))
```

### 函数

Makefile 中有很多内建函数。以下是一些常见的内建函数及其功能：

- \$(subst from, to, text)：在 text 中替换 from 为 to。
  STR := replaceAwithB
  RESULT := \$(subst A, B, $(STR))

  RESULT 的值将是 "replBceAwithB"

- \$(patsubst pattern, replacement, text)：对 text 进行模式替换，将符合 pattern 的字符串替换为 replacement。
  SOURCES := file1.c file2.c file3.c
  OBJECTS := \$(patsubst %.c,%.o,$(SOURCES))

  OBJECTS 的值将是 "file1.o file2.o file3.o"

  

- \$(strip string)：删除 string 首尾的空白字符（空格和制表符）。
  STR =     This is a string
  RESULT := \$(strip \$(STR))
  RESULT 的值将是 "This is a string"

  

- \$(findstring find, in)：在字符串 in 中查找子串 find，如果找到，返回 find，否则返回空字符串。
  IFDEF_TEST := \$(findstring _TEST_, $(DEFINES))
  $(filter pattern, text)：从 text 中选择符合 pattern 的字符串。支持 ? 和 % 通配符。
  FILES := a.c b.h c.cpp d.hpp
  C_FILES :=\\$(filter \%.c \%.cpp, \$(FILES))
  C_FILES 的值将是 "a.c c.cpp"

  

- \$(filter-out pattern, text)：从 text 中排除符合 pattern 的字符串。支持 ? 和 % 通配符。
  FILES := a.c b.h c.cpp d.hpp
  NOT_C_FILES := \$(filter-out %.c %.cpp, $(FILES))

  NOT_C_FILES 的值将是 "b.h d.hpp"

  

- \$(sort list)：对 list 中的单词进行排序并删除重复项。
  WORDS := Z A A C B Y
  UNIQUE_SORTED :=\ $(sort $(WORDS))
  UNIQUE_SORTED 的值将是 "A B C Y Z"

  

- \$(dir names)：返回 names 中各文件的目录部分，包括最后的斜杠。
  FILES := src/a.c include/b.h
  DIRS := \$(dir $(FILES))
  DIRS 的值将是 "src/ include/"

  

- \$(wildcard pattern)：返回符合 pattern 的文件列表。
  C_FILES := \$(wildcard *.c)

  

这只是 Makefile 中内建函数的一部分。更多函数和详细使用方法，可以查阅 GNU Make 的官方文档：
https://www.gnu.org/software/make/manual/make.html