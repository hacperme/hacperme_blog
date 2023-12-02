---
title: "riscv32-elf-gcc 在 wsl 上出现 Segmentation fault"
date: 2023-12-02T21:41:22+08:00
lastmod: 2023-12-02T21:41:22+08:00
author: ["hacper"]
tags:
    - BK7256
    - riscv32-elf-gcc
    - wsl
categories:
    - 笔记
description: "解决 riscv32-elf-gcc 在 wsl 上运行出现 Segmentation fault 问题" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
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


BK7256 的开发环境需要用到 RISCV 工具链，下载官方的工具链 [toolchain_v5.2.1.tar.gz](https://dl.bekencorp.com/tools/toolchain/riscv/toolchain_v5.2.1.tar.gz), 在 WSL 下编译 C 源码的时候出现 Segmentation fault 错误，运行不了。后面找到原因是 vsyscall 没开启导致的，后面遇到类似问题可以往这个方向排查。

WSL 开启 vsyscall 的方法如下：

用户根目录创建配置文件 .wslconfig，在 wsl2 下添加 `kernelCommandLine = vsyscall=emulate`

```yml
[wsl2]
kernelCommandLine = vsyscall=emulate
```

## 参考资料

- [Advanced settings configuration in WSL](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)
- [riscv32-elf-gcc在archlinux上直接Segmentation fault](https://bbs.archlinuxcn.org/viewtopic.php?id=13753)