---
title: "Rust 学习笔记之 Rust 数据结构"
date: 2024-10-18T01:18:13+08:00
lastmod: 2024-10-18T01:18:13+08:00
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

## 数组

```rust
fn main() {
    let mut a: [i8; 10] = [42; 10];
    a[5] = 0;
    println!("a: {a:#?}");
}
// let mut a: [i8; 10]  类型+长度， 后面是使用字面值复制
```

数组元素的访问和迭代：

```rust
fn main() {
    let primes = [2, 3, 5, 7, 11, 13, 17, 19];
    for prime in primes {
        for i in 2..prime {
            assert_ne!(prime % i, 0);
        }
    }
}
```

数组嵌套

```rust
let array = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];

```

## 元组

```rust
fn main() {
    let t: (i8, bool) = (7, true);
    println!("t.0: {}", t.0);
    println!("t.1: {}", t.1);
}
```

元组将不同类似的值组合成一个复合结构。

## 解构
从复合类型数据直接取值的语法：
```rust
fn print_tuple(tuple: (i32, i32)) {
    let (left, right) = tuple;
    println!("left: {left}, right: {right}");
}
```


## 引用

使用 `&`创建变量的引用，`*`解引用。
- 共享引用，不能修改其引用变量值，所有权不变化。
```rust
fn main() {
    let a = 'A';
    let b = 'B';
    let mut r: &char = &a;
    println!("r: {}", *r);
    r = &b;
    println!("r: {}", *r);
}
```
- 独占引用
它们的类型为 &mut T，在独占引用存在期间，不允许同时存在其他引用（无论是共享引用还是独占引用），并且无法访问引用的值。

```rust
fn main() {
    fn main() {
    let mut point = (1, 2);
    let x_coord = &mut point.0;
    *x_coord = 20;
    println!("point: {point:?}");
}

```

## 切片(slice)

切片从被切片的类型中借用 (borrow) 数据。
```rust
fn main() {
    let mut a: [i32; 6] = [10, 20, 30, 40, 50, 60];
    println!("a: {a:?}");

    let s: &[i32] = &a[2..4];

    println!("s: {s:?}");
}
```
&a[0..a.len()] and &a[..a.len()] are identical.

## 字符串

String is an owned, heap-allocated buffer of UTF-8 bytes.
&str is a slice of UTF-8 encoded bytes, similar to &[u8].

```rust
fn main() {
    let s1: &str = "World";
    println!("s1: {s1}");

    let mut s2: String = String::from("Hello ");
    println!("s2: {s2}");
    s2.push_str(s1);
    println!("s2: {s2}");

    let s3: &str = &s2[s2.len() - s1.len()..];
    println!("s3: {s3}");
}
```

## 结构体

定义、初始化、数据访问

```rust
struct Person {
    name: String,
    age: u8,
}

fn describe(person: &Person) {
    println!("{} is {} years old", person.name, person.age);
}

fn main() {
    let mut peter = Person { name: String::from("Peter"), age: 27 };
    describe(&peter);

    peter.age = 28;
    describe(&peter);

    let name = String::from("Avery");
    let age = 39;
    let avery = Person { name, age };
    describe(&avery);

    let jackie = Person { name: String::from("Jackie"), ..avery };
    describe(&jackie);
}
```

元组结构体, 没有成员字段名；

```rust
struct Point(i32, i32);

fn main() {
    let p = Point(17, 23);
    println!("({}, {})", p.0, p.1);
}

// 单字段封装容器（称为 newtype），如需对基元类型中的值的额外信息进行编码，使用 newtype 是一种非常好的方式
struct PoundsOfForce(f64);
struct Newtons(f64);

fn compute_thruster_force() -> PoundsOfForce {
    todo!("Ask a rocket scientist at NASA")
}

fn set_thruster_force(force: Newtons) {
    // ...
}

fn main() {
    let force = compute_thruster_force();
    set_thruster_force(force);
}
```