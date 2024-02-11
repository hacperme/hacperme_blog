---
title: "mklittlefs: littlefs 镜像打包和解析工具"
date: 2024-02-11T10:25:30+08:00
lastmod: 2024-02-11T10:25:30+08:00
author: ["hacper"]
tags:
    - mklittlefs
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "littlefs 镜像打包和解析工具" # 文章简单描述，会展示在主页
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
# mermaid: true

---


基于 [mklittlefs: https://github.com/earlephilhower/mklittlefs)](https://github.com/earlephilhower/mklittlefs) ，修改并增加了一些配置参数。

由于littlefs v1.x 和 v2.x 不兼容，编译了两个版本：

- mklittlefs-v1.exe 用于 littlefs v1.x.x

- mklittlefs-v2.exe 用于 littlefs v2.x.x

二者使用的命令参数一样。

## 命令参数

```c
mklittlefs-v1 or mklittlefs-v2          {-c <pack_dir>|-u <dest_dir>|-l} 
                                        [-T <from_file>] [-d <0-5>] [-a] [-w <number>] 
                                        [-r <number>] [-b <number>] [-s <number>] [--] 
                                        [--version] [-h] <image_file>

Where:

   -c <pack_dir>,  --create <pack_dir>
     (OR required)  create littlefs image from a directory
         -- OR --
   -u <dest_dir>,  --unpack <dest_dir>
     (OR required)  unpack littlefs image to a directory
         -- OR --
   -l,  --list
     (OR required)  list files in littlefs image


   -T <from_file>,  --from-file <from_file>
     when creating an image, include paths in from_file instead of scanning
     pack_dir

   -d <0-5>,  --debug <0-5>
     Debug level. 0 means no debug output.

   -a,  --all-files
     when creating an image, include files which are normally ignored;
     currently only applies to '.DS_Store' files and '.git' directories

   -w <number>,  --prosize <number>
     fs pro size, in bytes

   -r <number>,  --readsize <number>
     fs read_size, in bytes

   -b <number>,  --block <number>
     fs block size, in bytes

   -s <number>,  --size <number>
     fs image size, in bytes

   --,  --ignore_rest
     Ignores the rest of the labeled arguments following this flag.

   --version
     Displays version information and exits.

   -h,  --help
     Displays usage information and exits.

   <image_file>
     (required)  littlefs image file
```

`-c`: 制作镜像

`-u` 解析镜像

`-l` 列出镜像文件中的文件

其他参数：`read_size`,`prog_size`,`block_size` 根据 lfs_mount 挂载文件系统时 lfs_config 中的配置参数来填写。



## 使用示例

v1 版本

```bash
❯ .\mklittlefs-v1.exe --version
mklittlefs ver. 3.2.0-7-g1a3999a
Build configuration name: generic
LittleFS ver. v1.7.2
Extra build flags: (none)
LittleFS configuration:
  LFS_NAME_MAX: 256
  LFS_FILE_MAX: 2147483647

## 解析
❯ .\mklittlefs-v1.exe -u test3  -b 4096 -w 4096 -s 1048576 -r 4096 customer_fs.bin
Countdown.mp3    > ./test3/Countdown.mp3        size: 14671 Bytes
Loudwarn.mp3     > ./test3/Loudwarn.mp3 size: 46086 Bytes
Overspeed.mp3    > ./test3/Overspeed.mp3        size: 6313 Bytes
PowerOff.mp3     > ./test3/PowerOff.mp3 size: 33954 Bytes
PowerOn.mp3      > ./test3/PowerOn.mp3  size: 24355 Bytes
Register.mp3     > ./test3/Register.mp3 size: 23030 Bytes
Warn.mp3         > ./test3/Warn.mp3     size: 20942 Bytes
1K0dB10s.mp3     > ./test3/1K0dB10s.mp3 size: 159077 Bytes

## 打包
❯ .\mklittlefs-v1.exe -c test  -b 4096 -w 4096 -s 1048576 -r 4096 customer_fs.bin
/bt/ccc/341cf0fc5b670/0
/bt/cf/341cf0fc5b670/0
/bt/hash
/bt/keys/341cf0fc5b670/0
/bt/name
/bt/sc/341cf0fc5b670/0
/btps.db

## 查看镜像文件
❯ .\mklittlefs-v1.exe -l  -b 4096 -w 4096 -s 1048576 -r 4096 customer_fs.bin
<dir>   /bt
<dir>   /bt/ccc
<dir>   /bt/ccc/341cf0fc5b670
4       /bt/ccc/341cf0fc5b670/0
<dir>   /bt/cf
<dir>   /bt/cf/341cf0fc5b670
1       /bt/cf/341cf0fc5b670/0
16      /bt/hash
<dir>   /bt/keys
<dir>   /bt/keys/341cf0fc5b670
124     /bt/keys/341cf0fc5b670/0
13      /bt/name
<dir>   /bt/sc
<dir>   /bt/sc/341cf0fc5b670
4       /bt/sc/341cf0fc5b670/0
6144    /btps.db
```

v2 版本

```bash
❯  .\mklittlefs-v2.exe --version
mklittlefs ver. 3.2.0-7-g1a3999a
Build configuration name: generic
LittleFS ver. v2.5.1
Extra build flags: (none)
LittleFS configuration:
  LFS_NAME_MAX: 256
  LFS_FILE_MAX: 2147483647
  LFS_ATTR_MAX: 1022
      
## 解析
❯ .\mklittlefs-v2.exe -u test  -b 4096 -s 2097152 littlefs_2M.bin
btps.db  > ./test/btps.db       size: 6144 Bytes
bt//ccc//341cf0fc5b670/0         > ./test/bt/ccc/341cf0fc5b670/0        size: 4 Bytes
bt//cf//341cf0fc5b670/0  > ./test/bt/cf/341cf0fc5b670/0 size: 1 Bytes
bt/hash  > ./test/bt/hash       size: 16 Bytes
bt//keys//341cf0fc5b670/0        > ./test/bt/keys/341cf0fc5b670/0       size: 124 Bytes
bt/name  > ./test/bt/name       size: 13 Bytes
bt//sc//341cf0fc5b670/0  > ./test/bt/sc/341cf0fc5b670/0 size: 4 Bytes

## 打包
❯ .\mklittlefs-v2.exe -c test3  -b 4096 -s 2097152 littlefs_2M.bin
/1K0dB10s.mp3
/bt/ccc/341cf0fc5b670/0
/bt/cf/341cf0fc5b670/0
/bt/hash
/bt/keys/341cf0fc5b670/0
/bt/name
/bt/sc/341cf0fc5b670/0
/btps.db
/Countdown.mp3
/Loudwarn.mp3
/Overspeed.mp3
/PowerOff.mp3
/PowerOn.mp3
/Register.mp3
/Warn.mp3

## 列举镜像中的文件
❯ .\mklittlefs-v2.exe -l  -b 4096 -s 2097152 littlefs_2M.bin
159077  /1K0dB10s.mp3   Sun Feb 04 03:08:21 2024
14671   /Countdown.mp3  Sun Feb 04 03:08:21 2024
46086   /Loudwarn.mp3   Sun Feb 04 03:08:21 2024
6313    /Overspeed.mp3  Sun Feb 04 03:08:21 2024
33954   /PowerOff.mp3   Sun Feb 04 03:08:21 2024
24355   /PowerOn.mp3    Sun Feb 04 03:08:21 2024
23030   /Register.mp3   Sun Feb 04 03:08:21 2024
20942   /Warn.mp3       Sun Feb 04 03:08:21 2024
6144    /btps.db        Sun Feb 04 03:09:58 2024
<dir>   /bt     Sun Feb 04 03:09:58 2024
<dir>   /bt/ccc Sun Feb 04 03:09:58 2024
<dir>   /bt/ccc/341cf0fc5b670   Sun Feb 04 03:09:58 2024
4       /bt/ccc/341cf0fc5b670/0 Sun Feb 04 03:09:58 2024
<dir>   /bt/cf  Sun Feb 04 03:09:58 2024
<dir>   /bt/cf/341cf0fc5b670    Sun Feb 04 03:09:58 2024
1       /bt/cf/341cf0fc5b670/0  Sun Feb 04 03:09:58 2024
16      /bt/hash        Sun Feb 04 03:09:58 2024
<dir>   /bt/keys        Sun Feb 04 03:09:58 2024
<dir>   /bt/keys/341cf0fc5b670  Sun Feb 04 03:09:58 2024
124     /bt/keys/341cf0fc5b670/0        Sun Feb 04 03:09:58 2024
13      /bt/name        Sun Feb 04 03:09:58 2024
<dir>   /bt/sc  Sun Feb 04 03:09:58 2024
<dir>   /bt/sc/341cf0fc5b670    Sun Feb 04 03:09:58 2024
4       /bt/sc/341cf0fc5b670/0  Sun Feb 04 03:09:58 2024
Creation time:  Sun Feb 04 03:14:12 2024
```


## 编译

源码编译工具使用 xmake 和 gcc.
```bash
$ git clone git@github.com:hacperme/mklittlefs.git
$ git submodule update --init

$ xmake

# on windows, use xmake and MinGW

$ xmake f -p mingw --sdk=E:\workspace\tools\winlibs-x86_64-posix-seh-gcc-13.2.0-mingw-w64msvcrt-11.0.1-r2\mingw64
$ xmake
```
