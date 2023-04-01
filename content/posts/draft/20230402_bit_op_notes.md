---
title: "位运算笔记"
date: 2023-04-02T00:44:25+08:00
lastmod: 2023-04-02T00:44:25+08:00
author: ["hacper"]
tags:
    - 位运算
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
# cover:
#     image: ""
#     caption: ""
#     alt: ""
#     relative: false

---



## 判断奇偶性

```c
if ((x & 1) == 0) {
  x is even
}
else {
  x is odd
}
```

## 判断第n位是否是 1

```c
if (x & (1<<n)) {
  n-th bit is set
}
else {
  n-th bit is not set
}
```

## 第 n 位置 1

```c
y = x | (1<<n)
```

## 第 n 位置 0

```c
y = x & ~(1<<n)
```

## 翻转第 n 位

```c
y = x ^ (1<<n)
```

## 清除右边的第一个非 0 位

```c
y = x & (x-1)
```

## 仅保留右边的第一个非 0 位

```c
y = x & (-x)
```

## 将右边第一个非 0 位的后面全部置 1

```c
y = x | (x-1)
```

## 将右边第一个 0 位置 1，其余位置 0

```c
y = ~x & (x+1)
```

## 将右边第一个 0 位置 1

```c
y = x | (x+1)
```

