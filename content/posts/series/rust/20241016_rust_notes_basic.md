---
title: "Rust 学习笔记之Rust 开发环境"
date: 2024-10-16T00:46:29+08:00
lastmod: 2024-10-16T00:46:29+08:00
author: ["hacper"]
tags:
    - Rust
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

## Rust 安装

按照 rust [官网](https://www.rust-lang.org/learn/get-started)指导进行安装，注意 windows 系统下需要安装 [Visual Studio C++ Build tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)。

windows 下支持两套工具链，MSVC 和 GNU，但 windows 系统环境大多数库使用 MSVC 编译，MSVC 是 windows 下的默认工具链。

## Rust 开发环境及其开发工具 

- cargo
    Rust 依赖项管理器和构建工具
- rustc
    Rust 编译器
- rustup
    Rust 工具链安装和更新工具


## 开发工具的使用

- cargo 的使用：创建、编译、运行、测试

```bash
    # 常用指令
    cargo new
    cargo run
    cargo bulid
    cargo build --release
    cargo check
```


- rustup 的使用：新增工具链、更新、默认工具链切换

- vscode 插件

