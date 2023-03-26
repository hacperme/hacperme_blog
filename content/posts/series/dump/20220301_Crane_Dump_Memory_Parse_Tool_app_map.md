---
title: "Quec_Crane_Dump_Memory_Parse_Tool_v1.1.2 增加 app.map 文件解析的修改方法"
date: 2022-03-01T00:40:42+08:00
lastmod: 2023-03-27T00:40:42+08:00
author: ["hacper"]
tags:
    - dump
    - ASR
    - python
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "为 dump 解析工具增加 app.map 文件支持。" # 文章简单描述，会展示在主页
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
# cover:
#     image: ""
#     caption: ""
#     alt: ""
#     relative: false

---

Quec_Crane_Dump_Memory_Parse_Tool_v1.1.2 是一个使用 python 写的，解析ASR 平台 dump 的一个工具。通常在使用 trace 32 工具分析dump的时候，也结合 Quec_Crane_Dump_Memory_Parse_Tool 工具辅助分析。

Quec_Crane_Dump_Memory_Parse_Tool 只能根据 kernel.map 解析dump，而在app侧编译的app.map无法解析。kernel.map 是在 armcc 工具套件下编译生成的，而 app.map 则使用的是 armgcc 编译工具，二者生成的map文件格式有较大差异，Quec_Crane_Dump_Memory_Parse_Tool 没有支持 armgcc 

生成的map文件，所以通过Quec_Crane_Dump_Memory_Parse_Too 解析的结果只包含kernel侧的信息，通常客户的代码是在 app 侧，如果问题发生在app侧，在分析时无法通过app侧的解析得到更多线索，这是这个工具存在的问题。



## 修改以支持app.map解析的方法

主要修改两个文件：_scripts/cp_dump.py 和 _scripts/cp_map.py

_scripts/cp_dump.py 里面去除只能解析一个map文件的限制，直接返回包含app.map和kernel.map这两个的文件的路径：

```diff
diff --git a/_scripts/cp_dump.py b/_scripts/cp_dump.py
index 76221d4..3b6920b 100644
--- a/_scripts/cp_dump.py
+++ b/_scripts/cp_dump.py
@@ -79,12 +79,12 @@ class CpDump:
                 print("no map file found!")
                 print('================================\n')
                 sys.exit(2)
-        if len(map_list) > 1:
-            print("too many map files. Which one to be used?\n")
-            for ff in map_list:
-                print("\t"+ff+"\n")
-            sys.exit(3)
-        return map_list[0]
+        # if len(map_list) > 1:
+        #     print("too many map files. Which one to be used?\n")
+        #     for ff in map_list:
+        #         print("\t"+ff+"\n")
+        #     sys.exit(3)
+        return map_list
     
     def memory_block(self, address):
         if address >= self.dtcm_base and address < self.dtcm_base + len(self.dtcm):
```

在 _scripts/cp_map.py 这个文件里面，分别加载kernel.map和app.map文件，并新增对 app.map 解析的方法，主要新增一个正则表达式用来提取app.map中的函数名、函数起始地址和函数的大小：

```diff
iff --git a/_scripts/cp_map.py b/_scripts/cp_map.py
index 6f07124..f99bc25 100644
--- a/_scripts/cp_map.py
+++ b/_scripts/cp_map.py
@@ -7,10 +7,30 @@ class ImageMap:
         self.gd_list = []
         self.ddr_base = 0
         self.dtcm_base = 0
-        if os.path.isfile(fname):
-            self.load_map_file(fname)
+        app_map = ""
+        kernel_map = ""
+        map_len = len(fname)
+        if map_len > 1:
+            for ff in fname:
+                if "app" in ff:
+                    app_map = ff
+                    print("INFO: find app map {}\n".format(app_map))
+                else:
+                    kernel_map = ff
+                    print("INFO: find kernel map {}\n".format(kernel_map))
+                
         else:
-            print("ERROR: %s not exist\n", fname)
+
+            kernel_map = fname[0]
+        if os.path.isfile(kernel_map):
+            self.load_kernel_map_file(kernel_map)
+        else:
+            print("ERROR: kernel_map:{} not exist\n".format(kernel_map))
+
+        if os.path.isfile(app_map):
+            self.load_app_map_file(app_map)
+        else:
+            print("ERROR: app_map:{} not exist\n".format(app_map))
         
     def get_thumb_function(self, line):
         r = re.search('\s+(.+)\s+0x([0-9a-fA-F]+)\s+Thumb Code\s+(\d+)\s+\S+\S+', line)
@@ -21,6 +41,20 @@ class ImageMap:
         fn['start'] = int(txt_start, 16)
         fn['size'] = int(txt_size)
         return fn
+    
+    def get_app_function(self, map_data):
+        fn = []
+        funcs = re.findall('\s+\.text\.(\w.*?)\n\s.*?0x(?!00000000)([0-9a-fA-F]+)\s+0x([0-9a-fA-F]+)(?:.*?\.o)', map_data)
+        if len(funcs)==0:
+            print("ERROR:no app funcs found")
+            return fn
+        keys = ["name", "start", "size"]
+        for i in funcs:
+            f = dict(zip(keys, [i[0],int(i[1], 16),int(i[2], 16)]))
+            fn.append(f)
+        
+        return fn
+
         
     def get_arm_function(self, line):    
         r = re.search('\s+(.+)\s+0x([0-9a-fA-F]+)\s+ARM Code\s+(\d+)\s+\S+\S+', line)
@@ -54,7 +88,7 @@ class ImageMap:
         txt_name, txt_start, txt_size = r.groups()
         return int(txt_start, 16)
 
-    def load_map_file(self, fname):
+    def load_kernel_map_file(self, fname):
         self.fn_list = []
         self.gd_list = []
         self.ddr_base = 0
@@ -72,7 +106,15 @@ class ImageMap:
                 elif line.find('Image$$DDR_DTCM$$Base') > 0:
                     self.dtcm_base = self.get_dtcm_base(line)
         self.fn_list.sort(key=lambda x:x['start'])
-        
+
+    def load_app_map_file(self, fname):
+        with open(fname, 'r') as f:
+            map_data = f.read()
+        fn_list = self.get_app_function(map_data)
+        self.fn_list.extend(fn_list)
+        self.fn_list.sort(key=lambda x:x['start']) 
+        # with open("all_funcs.txt", 'w') as f:
+        #     print(self.fn_list,file=f)
     
     def search_functions(self, func_list, address):
         if len(func_list) > 10:
```

其中提取app.map中函数信息的正则表达式: `\s+\.text\.(\w.*?)\n\s.*?0x(?!00000000)([0-9a-fA-F]+)\s+0x([0-9a-fA-F]+)(?:.*?\.o)`

提取函数名、起始地址和函数大小，有这三个信息之后就可以根据内存地址找到对应的函数了。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.5zw699c7x7w0.png)



![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.7gdw7w6kjy80.png)

## 结果对比

修改之后，output_all_task.txt 解析任务栈中函数调用关系的结果中，补充了app侧的函数调用。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.16fe13kbvn0g.png)



![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.6ovxrht33bk0.png)

output_osa_mem.txt 中也增加了在app侧申请内存的函数的显示。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.natwx27pd40.png)



![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/xxx.3ktys22a8mc0.png)

## Quec_Crane_Dump_Memory_Parse_Tool 工具的使用场景

1. 排查内存泄漏问题和内存破坏问题

   通过 output_osa_mem.txt 的解析结果定位内存泄漏和内存破坏的位置，以及内存申请失败问题。

2. 排查中断、高优先级task 耗时长导致死机问题

   output_rti_tsk.txt 解析结果记录了一段时间内任务调度的情况，其中包含每个task执行的耗时时间，从中可以判断和定位一些中断回调、高优先级task有阻塞动作导致的死机问题

3. 查看死机时task的调用栈信息

   output_all_task.txt 里面包含死机时函数的调用栈信息，在 trace 32 解析dump时看不到任务调用栈信息的情况下（比如 pc寄存器非法，调用栈为空的情况），这时可以通过output_all_task.txt看调用栈信息，可以定位出现问题的大致范围。

4. 作为一种调试方法

   主动触发dump，通过 output_rti_tsk.txt 查看任务切换的情况，调试多任务调度下的一些奇怪问题。

   通过 output_all_task.txt， 除了可以查看当前死机的task，还可以查看其他task的信息，任务的优先级、任务栈大小和剩余大小和任务调用栈，通过这些信息也可以看下这些task的优先级和任务栈大小设置是否合理，以及task在休眠之前调用了哪些函数。

