---
title: "使用 valgrind 检测内存问题示例"
date: 2023-08-19T00:38:26+08:00
lastmod: 2023-08-19T00:38:26+08:00
author: ["hacper"]
tags:
    - valgrind
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
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

## 介绍

valgrind 是一套工具集，常用于检测软件内存泄漏和分析软件性能问题。本文只记录使用 valgrind 检测内存相关问题的示例。

## valgrind 使用

使用 valgrind 调用将要测试的程序，加上参数 `--leak-check=full --show-leak-kinds=all`, 如下面示例，检测 test 程序有没有问题。
```bash
valgrind --leak-check=full --show-leak-kinds=all ./test
```

## 常见内存问题

1. 内存泄漏

```c
void test_memory_leak(void)
{
    char *test_data = NULL;
    test_data = malloc(1024);
    memset(test_data, 0, 1024);
    sprintf(test_data, "[test]\r\nkey=123\r\n");
    printf("test_data:%s\r\n", test_data);
}
```

```bash
valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
==344== Memcheck, a memory error detector
==344== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==344== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==344== Command: ./build/build_out/target/test
==344==
test_data:[test]
key=123

test count:36, test passed:36, 100.00%
==344== 
==344== HEAP SUMMARY:
==344==     in use at exit: 1,024 bytes in 1 blocks
==344==   total heap usage: 181 allocs, 180 frees, 28,748 bytes allocated
==344==
==344== 1,024 bytes in 1 blocks are definitely lost in loss record 1 of 1
==344==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==344==    by 0x10A59C: test_memory_leak (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
==344==    by 0x10A600: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
==344==    by 0x10A61B: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
==344==
==344== LEAK SUMMARY:
==344==    definitely lost: 1,024 bytes in 1 blocks
==344==    indirectly lost: 0 bytes in 0 blocks
==344==      possibly lost: 0 bytes in 0 blocks
==344==    still reachable: 0 bytes in 0 blocks
==344==         suppressed: 0 bytes in 0 blocks
==344==
==344== For lists of detected and suppressed errors, rerun with: -s
==344== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

2. 内存越界

   写越界

   ```c
   void test_memory(void)
   {
       // 内存越界示例
       char *test_data = NULL;
       test_data = malloc(10);
       memset(test_data, 0, 10);
       strcpy(test_data, "test");
       test_data[10] = '\0';
       free(test_data);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==56== Memcheck, a memory error detector
   ==56== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==56== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==56== Command: ./build/build_out/target/test
   ==56==
   ==56== Invalid write of size 1
   ==56==    at 0x10A5AD: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==56==    by 0x10A5D5: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==56==    by 0x10A5F0: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==56==  Address 0x4a52c3a is 0 bytes after a block of size 10 alloc'd
   ==56==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)       
   ==56==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==56==    by 0x10A5D5: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==56==    by 0x10A5F0: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==56==
   test count:36, test passed:36, 100.00%
   ==56==
   ==56== HEAP SUMMARY:
   ==56==     in use at exit: 0 bytes in 0 blocks
   ==56==   total heap usage: 181 allocs, 181 frees, 27,734 bytes allocated
   ==56==
   ==56== All heap blocks were freed -- no leaks are possible
   ==56==
   ==56== For lists of detected and suppressed errors, rerun with: -s
   ==56== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
   ```

   读越界

   ```c
   void test_memory(void)
   {
       char *test_data = NULL;
       test_data = malloc(10);
       memset(test_data, 0, 10);
       strcpy(test_data, "test");
       printf("test_data:%c\r\n", test_data[10]);
       free(test_data);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==118== Memcheck, a memory error detector
   ==118== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==118== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==118== Command: ./build/build_out/target/test
   ==118==
   ==118== Invalid read of size 1
   ==118==    at 0x10A5AD: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==118==    by 0x10A5EB: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==118==    by 0x10A606: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==118==  Address 0x4a52c3a is 0 bytes after a block of size 10 alloc'd
   ==118==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==118==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==118==    by 0x10A5EB: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==118==    by 0x10A606: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==118==
   test_data:
   test count:36, test passed:36, 100.00%
   ==118== 
   ==118== HEAP SUMMARY:
   ==118==     in use at exit: 0 bytes in 0 blocks
   ==118==   total heap usage: 181 allocs, 181 frees, 27,734 bytes allocated
   ==118==
   ==118== All heap blocks were freed -- no leaks are possible
   ==118==
   ==118== For lists of detected and suppressed errors, rerun with: -s
   ==118== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
   ```

   

3. double free

   ```c
   void test_memory(void)
   {
       char *test_data = NULL;
       test_data = malloc(10);
       memset(test_data, 0, 10);
       strcpy(test_data, "test");
       free(test_data);
       free(test_data);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==177== Memcheck, a memory error detector
   ==177== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==177== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==177== Command: ./build/build_out/target/test
   ==177==
   ==177== Invalid free() / delete / delete[] / realloc()
   ==177==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==177==    by 0x10A5BC: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==    by 0x10A5D6: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==    by 0x10A5F1: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==  Address 0x4a52c30 is 0 bytes inside a block of size 10 free'd
   ==177==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==177==    by 0x10A5B0: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==    by 0x10A5D6: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==    by 0x10A5F1: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==  Block was alloc'd at
   ==177==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==177==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==    by 0x10A5D6: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==    by 0x10A5F1: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==177==
   test count:36, test passed:36, 100.00%
   ==177== 
   ==177== HEAP SUMMARY:
   ==177==     in use at exit: 0 bytes in 0 blocks
   ==177==   total heap usage: 181 allocs, 182 frees, 27,734 bytes allocated
   ==177==
   ==177== All heap blocks were freed -- no leaks are possible
   ==177==
   ==177== For lists of detected and suppressed errors, rerun with: -s
   ==177== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
   ```

   

4. use after free

   ```c
   void test_memory(void)
   {
       char *test_data = NULL;
       test_data = malloc(10);
       memset(test_data, 0, 10);
       strcpy(test_data, "test");
       free(test_data);
       printf("test_data:%s\r\n", test_data);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==330== Memcheck, a memory error detector
   ==330== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==330== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==330== Command: ./build/build_out/target/test
   ==330==
   ==330== Invalid read of size 1
   ==330==    at 0x483EF46: strlen (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x48CCD14: __vfprintf_internal (vfprintf-internal.c:1688)
   ==330==    by 0x48B5D3E: printf (printf.c:33)
   ==330==    by 0x10A5C8: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Address 0x4a52c30 is 0 bytes inside a block of size 10 free'd
   ==330==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A5B0: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Block was alloc'd at
   ==330==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==
   ==330== Invalid read of size 1
   ==330==    at 0x483EF54: strlen (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x48CCD14: __vfprintf_internal (vfprintf-internal.c:1688)
   ==330==    by 0x48B5D3E: printf (printf.c:33)
   ==330==    by 0x10A5C8: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Address 0x4a52c31 is 1 bytes inside a block of size 10 free'd
   ==330==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A5B0: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Block was alloc'd at
   ==330==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==
   ==330== Invalid read of size 1
   ==330==    at 0x48E370C: _IO_new_file_xsputn (fileops.c:1219)
   ==330==    by 0x48E370C: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1197)
   ==330==    by 0x48CB0FB: __vfprintf_internal (vfprintf-internal.c:1688)
   ==330==    by 0x48B5D3E: printf (printf.c:33)
   ==330==    by 0x10A5C8: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Address 0x4a52c33 is 3 bytes inside a block of size 10 free'd
   ==330==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A5B0: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Block was alloc'd at
   ==330==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==
   ==330== Invalid read of size 1
   ==330==    at 0x48436A0: mempcpy (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x48E3631: _IO_new_file_xsputn (fileops.c:1236)
   ==330==    by 0x48E3631: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1197)
   ==330==    by 0x48CB0FB: __vfprintf_internal (vfprintf-internal.c:1688)
   ==330==    by 0x48B5D3E: printf (printf.c:33)
   ==330==    by 0x10A5C8: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Address 0x4a52c33 is 3 bytes inside a block of size 10 free'd
   ==330==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A5B0: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Block was alloc'd at
   ==330==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==
   ==330== Invalid read of size 1
   ==330==    at 0x48436B2: mempcpy (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x48E3631: _IO_new_file_xsputn (fileops.c:1236)
   ==330==    by 0x48E3631: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1197)
   ==330==    by 0x48CB0FB: __vfprintf_internal (vfprintf-internal.c:1688)
   ==330==    by 0x48B5D3E: printf (printf.c:33)
   ==330==    by 0x10A5C8: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Address 0x4a52c31 is 1 bytes inside a block of size 10 free'd
   ==330==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A5B0: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==  Block was alloc'd at
   ==330==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==330==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5E2: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==    by 0x10A5FD: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==330==
   test_data:test
   test count:36, test passed:36, 100.00%
   ==330== 
   ==330== HEAP SUMMARY:
   ==330==     in use at exit: 0 bytes in 0 blocks
   ==330==   total heap usage: 181 allocs, 181 frees, 27,734 bytes allocated
   ==330==
   ==330== All heap blocks were freed -- no leaks are possible
   ==330==
   ==330== For lists of detected and suppressed errors, rerun with: -s
   ==330== ERROR SUMMARY: 13 errors from 5 contexts (suppressed: 0 from 0)
   ```

   

5. free 不存在的地址

   ```c
   void test_memory(void)
   {
       char *test_data = NULL;
       test_data = malloc(10);
       memset(test_data, 0, 10);
       strcpy(test_data, "test");
       test_data++;
       free(test_data);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==395== Memcheck, a memory error detector
   ==395== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==395== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==395== Command: ./build/build_out/target/test
   ==395==
   ==395== Invalid free() / delete / delete[] / realloc()
   ==395==    at 0x483CA3F: free (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==395==    by 0x10A5B5: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==    by 0x10A5CF: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==    by 0x10A5EA: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==  Address 0x4a52c31 is 1 bytes inside a block of size 10 alloc'd
   ==395==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==395==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==    by 0x10A5CF: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==    by 0x10A5EA: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==
   test count:36, test passed:36, 100.00%
   ==395==
   ==395== HEAP SUMMARY:
   ==395==     in use at exit: 10 bytes in 1 blocks
   ==395==   total heap usage: 181 allocs, 181 frees, 27,734 bytes allocated
   ==395==
   ==395== 10 bytes in 1 blocks are definitely lost in loss record 1 of 1
   ==395==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==395==    by 0x10A57C: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==    by 0x10A5CF: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==    by 0x10A5EA: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==395==
   ==395== LEAK SUMMARY:
   ==395==    definitely lost: 10 bytes in 1 blocks
   ==395==    indirectly lost: 0 bytes in 0 blocks
   ==395==      possibly lost: 0 bytes in 0 blocks
   ==395==    still reachable: 0 bytes in 0 blocks
   ==395==         suppressed: 0 bytes in 0 blocks
   ==395==
   ==395== For lists of detected and suppressed errors, rerun with: -s
   ==395== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
   ```

6. 访问非法地址

   ```c
   static void test_parser(void)
   {
       test_parser_normal();
       // test abnormal
       test_parser_abnormal();
       test_memory();
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==480== Memcheck, a memory error detector
   ==480== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==480== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==480== Command: ./build/build_out/target/test
   ==480==
   ==480== Invalid write of size 1
   ==480==    at 0x10A577: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==480==    by 0x10A593: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==480==    by 0x10A5AE: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==480==  Address 0x23444 is not stack'd, malloc'd or (recently) free'd
   ==480==
   ==480==
   ==480== Process terminating with default action of signal 11 (SIGSEGV)
   ==480==  Access not within mapped region at address 0x23444
   ==480==    at 0x10A577: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==480==    by 0x10A593: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==480==    by 0x10A5AE: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==480==  If you believe this happened as a result of a stack
   ==480==  overflow in your program's main thread (unlikely but
   ==480==  possible), you can try to increase the size of the
   ==480==  main thread stack using the --main-stacksize= flag.
   ==480==  The main thread stack size used in this run was 8388608.
   ==480== 
   ==480== HEAP SUMMARY:
   ==480==     in use at exit: 0 bytes in 0 blocks
   ==480==   total heap usage: 179 allocs, 179 frees, 26,700 bytes allocated
   ==480==
   ==480== All heap blocks were freed -- no leaks are possible
   ==480==
   ==480== For lists of detected and suppressed errors, rerun with: -s
   ==480== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
   make: *** [Makefile:12: memcheck] Segmentation fault
   ```

   

7. 局部变量越界

   写越界

   ```c
   void test_memory(void)
   {
       char test_data[10] = {0};
       sprintf(test_data, "test333333333333333333");
       printf ("%s\r\n", test_data);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==527== Memcheck, a memory error detector
   ==527== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==527== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==527== Command: ./build/build_out/target/test
   ==527==
   test333333333333333333
   *** stack smashing detected ***: terminated
   ==527==
   ==527== Process terminating with default action of signal 6 (SIGABRT)
   ==527==    at 0x489700B: raise (raise.c:51)
   ==527==    by 0x4876858: abort (abort.c:79)
   ==527==    by 0x48E126D: __libc_message (libc_fatal.c:155)
   ==527==    by 0x4983AB9: __fortify_fail (fortify_fail.c:26)
   ==527==    by 0x4983A85: __stack_chk_fail (stack_chk_fail.c:24)
   ==527==    by 0x10A5EC: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==527==    by 0x10A605: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==527==    by 0x1FFEFFFE4F: ???
   ==527==    by 0x10A620: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==527==
   ==527== HEAP SUMMARY:
   ==527==     in use at exit: 0 bytes in 0 blocks
   ==527==   total heap usage: 180 allocs, 180 frees, 27,724 bytes allocated
   ==527==
   ==527== All heap blocks were freed -- no leaks are possible
   ==527==
   ==527== For lists of detected and suppressed errors, rerun with: -s
   ==527== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
   ```

   读越界

   ```c
   void test_memory(void)
   {
       char test_data[100] = {0};
       sprintf(test_data, "test333333333333333333");
       printf ("%c\r\n", test_data[100]);
   }
   ```

   ```bash
   valgrind --leak-check=full --show-leak-kinds=all ./build/build_out/target/test
   ==571== Memcheck, a memory error detector
   ==571== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
   ==571== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
   ==571== Command: ./build/build_out/target/test
   ==571==
   ==571== Conditional jump or move depends on uninitialised value(s)
   ==571==    at 0x48E4DDD: _IO_file_overflow@@GLIBC_2.2.5 (fileops.c:783)
   ==571==    by 0x48CD8ED: __vfprintf_internal (vfprintf-internal.c:1688)
   ==571==    by 0x48B5D3E: printf (printf.c:33)
   ==571==    by 0x10A632: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==    by 0x10A660: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==    by 0x10A67B: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==
   ==571== Syscall param write(buf) points to uninitialised byte(s)
   ==571==    at 0x4962077: write (write.c:26)
   ==571==    by 0x48E2E8C: _IO_file_write@@GLIBC_2.2.5 (fileops.c:1181)
   ==571==    by 0x48E4950: new_do_write (fileops.c:449)
   ==571==    by 0x48E4950: _IO_new_do_write (fileops.c:426)
   ==571==    by 0x48E4950: _IO_do_write@@GLIBC_2.2.5 (fileops.c:423)
   ==571==    by 0x48E36B4: _IO_new_file_xsputn (fileops.c:1244)
   ==571==    by 0x48E36B4: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1197)
   ==571==    by 0x48CAFE5: __vfprintf_internal (vfprintf-internal.c:1719)
   ==571==    by 0x48B5D3E: printf (printf.c:33)
   ==571==    by 0x10A632: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==    by 0x10A660: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==    by 0x10A67B: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==  Address 0x4a52c30 is 0 bytes inside a block of size 1,024 alloc'd
   ==571==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
   ==571==    by 0x48D5D03: _IO_file_doallocate (filedoalloc.c:101)
   ==571==    by 0x48E5ECF: _IO_doallocbuf (genops.c:347)
   ==571==    by 0x48E4F2F: _IO_file_overflow@@GLIBC_2.2.5 (fileops.c:745)
   ==571==    by 0x48CD8ED: __vfprintf_internal (vfprintf-internal.c:1688)
   ==571==    by 0x48B5D3E: printf (printf.c:33)
   ==571==    by 0x10A632: test_memory (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==    by 0x10A660: test_parser (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==    by 0x10A67B: main (in /mnt/d/workspace/lwiniparser/build/build_out/target/test)
   ==571==
   
   test count:36, test passed:36, 100.00%
   ==571== 
   ==571== HEAP SUMMARY:
   ==571==     in use at exit: 0 bytes in 0 blocks
   ==571==   total heap usage: 180 allocs, 180 frees, 27,724 bytes allocated
   ==571==
   ==571== All heap blocks were freed -- no leaks are possible
   ==571==
   ==571== Use --track-origins=yes to see where uninitialised values come from
   ==571== For lists of detected and suppressed errors, rerun with: -s
   ==571== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
   ```

   