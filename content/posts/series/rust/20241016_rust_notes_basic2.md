---
title: "Rust 学习笔记之 Rust 的类型和值"
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

## 变量

变量和值的绑定关系，可变变量和不可变变量。变量的值默认不可变，添加 mut 关键字才能修改变量的值。
变量遮蔽的理解。

```rust
fn main() {
    let a = 10;
    println!("before: {a}");
    {
        let a = "hello";
        println!("inner scope: {a}");

        let a = true;
        println!("shadowed in inner scope: {a}");
    }

    println!("after: {a}");
}
```
变量的作用域在 `{}` 内。变量遮蔽是指同名变量的情况，同名变量不同类型也可以存在。

## 基本数据类型

| 类型             | 字面量                                     |                                |
| ---------------- | ------------------------------------------ | ------------------------------ |
| 有符号整数       | `i8`、`i16`、`i32`、`i64`、`i128`、`isize` | `-10`、`0`、`1_000`、`123_i64` |
| 无符号整数       | `u8`、`u16`、`u32`、`u64`、`u128`、`usize` | `0`、`123`、`10_u16`           |
| 浮点数           | `f32`、`f64`                               | `3.14`、`-10.0e20`、`2_f32`    |
| Unicode 标量类型 | `char`                                     | `'a'`、`'α'`、`'∞'`            |
| 布尔值           | `bool`                                     | `true`、`false`                |

- `iN`, `uN` 和 `fN` 占用 *N* 位，
- `isize` 和 `usize` 占用一个指针大小的空间，
- `char` 占用 32 位空间，
- `bool` 占用 8 位空间。



数字中的所有下划线均可忽略，它们只是为了方便辨识。`1_000` 可以写为 `1000`（或 `10_00`），而 `123_i64` 可以写为 `123i64`。
当整数字面量的类型没有写确定类型时，Rust 默认为 i32。浮点字面量默认为 f64。

## 算术运算

## 类型推导

Rust 编译器可以根据变量声明和用法来推导其类型。