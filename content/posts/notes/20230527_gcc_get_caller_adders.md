---
title: "使用 gcc 内置接口获取当前函数的返回地址"
date: 2023-05-27T09:09:15+08:00
lastmod: 2023-05-27T09:09:15+08:00
author: ["hacper"]
tags:
    - gcc
    - c
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "c 代码中获取当前函数的返回地址方法" # 文章简单描述，会展示在主页
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

作为调试目的，有时候可能需要获取当前函数的调用者信息，比如在 malloc 申请一块内存的时候，记录 caller 的地址，当发生内存泄漏或者内存破坏时，可以使用 caller 地址大致定位是哪块代码有问题。

我们可以通过 GCC 的内置接口`void *` **__builtin_return_address** `(unsigned int level)` 获取当前函数的返回地址，也就是调用者。
参数 level 表示扫描的栈帧层级，0 表示返回当前函数的返回地址，1 便是当前函数的调用者的返回地址，以此类推。

测试代码

```c
void test_return(void)
{
	shell_printf("test return\r\n");
	void *p =  __builtin_return_address (0);
	shell_printf("1 %p\r\n", p);
	p = __builtin_extract_return_addr (__builtin_return_address (0));
	shell_printf("2 %p\r\n", p);
}

void test_return1(void)
{
	test_return();
	shell_printf("test return\r\n");
}

/**
 * @brief test command
 */
void shell_test_cmd(int argc, char *argv)
{
	unsigned int i;
	shell_printf("test command:\r\n");
	for (i = 0; i < argc; i++)
	{
		shell_printf("paras %d: %s\r\n", i, &(argv[(int)argv[i]]));
	}
	test_return1();
	void *p = test_return1;
	shell_printf("%p\r\n", p);
	
}

```



```bash
nr@root:test
test command:
paras 0: test
test return
1 004EADA6
2 004EADA6
test return
004EAFB0

```
004EADA6 是 test_return 的返回地址。004EAFB0 是直接打印的函数地址，是 test_return 的入口地址。然后通过 addr2line.exe 验证这两个地址。

```bash
D:\workspace\QT\qt_idf\examples\shell>D:\workspace\QT\qt_idf\tools\mingw32\bin\addr2line.exe -e D:/workspace/QT/qt_idf/examples/shell/build/build_out/target/shell.exe -f 004EADA6 004EAFB0
test_return1
D:/workspace/QT/qt_idf/examples/shell/cmds/cmds.c:56
test_return1
D:/workspace/QT/qt_idf/examples/shell/cmds/cmds.c:54
```

004EADA6 是 test_return1 中第 56 行代码位置，004EAFB0 是 test_return1 中第 54 行代码位置。
![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.2lbwjc2df7s0.webp)


另外，使用 addr2line.exe 还碰到出现 dwarf error: could not find abbrev number 108. 问题，原因是编译的时候没有打开调试信息，在编译配置中增加 -g3 即可解决。
