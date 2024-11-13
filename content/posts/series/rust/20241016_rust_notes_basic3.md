---
title: "Rust 学习笔记之 Rust 的控制流"
date: 2024-10-16T01:43:02+08:00
lastmod: 2024-10-16T01:43:02+08:00
author: ["hacper"]
tags:
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

## if 表达式

```rust
if x == 0 {
    println!("zero!");
} else if x < 100 {
    println!("biggish");
} else {
    println!("huge");
}
```

if 作为表达式的情况, 结束必须要 `;`

```rust
    let size = if x < 20 { "small" } else { "large" };
```

## 循环控制

- while

```rust
fn main() {
    let mut x = 200;
    while x >= 10 {
        x = x / 2;
    }
    println!("Final x: {x}");
}
```

- loop

```rust
fn main() {
    let mut i = 0;
    loop {
        i += 1;
        println!("{i}");
        if i > 100 {
            break;
        }
    }
}
```

- for

```rust
fn main() {
    for x in 1..5 {
        println!("x: {x}");
    }

    for elem in [1, 2, 3, 4, 5] {
        println!("elem: {elem}");
    }
}

// for x in 1..5 不包含边界情况，范围 1 - 4
// for x in 1..=5 迭代包含5
```

- break 和 continue

除了和其他编程语言类似的用法，continue 和 break 都可以选择接受一个标签参数，用来 终止嵌套循环：
```rust
fn main() {
    let s = [[5, 6, 7], [8, 9, 10], [21, 15, 32]];
    let mut elements_searched = 0;
    let target_value = 10;
    'outer: for i in 0..=2 {
        for j in 0..=2 {
            elements_searched += 1;
            if s[i][j] == target_value {
                break 'outer;
            }
        }
    }
    print!("elements searched: {elements_searched}");
}
```

## 函数

```rust
fn gcd(a: u32, b: u32) -> u32 {
    if b > 0 {
        gcd(b, a % b)
    } else {
        a
    }
}

fn main() {
    println!("gcd: {}", gcd(143, 52));
}
```

- fn 关键字定义函数
- 参数名在前，参数类型在后
- 返回值格式 -> 类型
- 结束没有; return，使用表达式的值作为返回值。

## 宏

常用宏：

- println!(format, ..) prints a line to standard output, applying formatting described in std::fmt.
- format!(format, ..) 的用法与 println! 类似，但它以字符串形式返回结果。
- dbg!(expression) 会记录表达式的值并返回该值。
- todo!() 用于标记尚未实现的代码段。如果执行该代码段，则会触发 panic。
- unreachable!() 用于标记无法访问的代码段。如果执行该代码段，则会触发 panic。