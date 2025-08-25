---
title: "FC41D heap 空间大小优化分析方法"
date: 2024-07-11T02:00:18+08:00
lastmod: 2024-07-14T02:00:18+08:00
author: ["hacper"]
tags:
   - FC41D
   - heap
   - gcc
   - freertos
   - bk7231m
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "通过链接器、map文件分析程序内存使用大小，优化内存布局。" # 文章简单描述，会展示在主页
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

内存优化是指 ram 或者 rom 的使用大小优化，想要优化内存，必须得了解下面这几个知识：

- 在程序中，得知道哪些类型的代码是占用 ram，哪些代码是存储在 rom 空间上的。
- 熟悉 ld 文件，知道程序的内存布局是怎样的，有哪些类型的内存、内存大小，不同的代码段（.text .rodata .data .bss）分别分配到哪个内存空间。
- 会看 map 文件，了解程序中内存使用的细节。



gcc 工具链中的链接器有一个功能，配置加上 -Wl,--print-memory-usage，可以在链接之后输出内存使用情况。

![](https://github.com/hacperme/picx-images-hosting/raw/master/20240711/image-20240711143848316.sytyorxq8.webp)

heap 内存放在 ram 这块内存，在.data .bss段内存之后，根据代码使用情况动态调整大小。从 ld 文件和 heap_4.c 可以看到 _empty_ram 就是 heap 的起始地址，结束地址到 ram 的末尾。

```ld
. = ORIGIN(ram);
/* globals.for example: int ram_data[3]={4,5,6}; */		/* VMA in RAM, but keep LMA in flash */
	_begin_data = .;
        .data :
	{
	    *(.data .data.*)
	    *(.sdata) 
	    *(.gnu.linkonce.d*)
    	SORT(CONSTRUCTORS)
        } >ram AT>flash
        _end_data = .;
	
	/* Loader will copy data from _flash_begin to _ram_begin..ram_end */
	_data_flash_begin = LOADADDR(.data);
	_data_ram_begin = ADDR(.data);
	_data_ram_end = .;

/* uninitialized data section - global   int i; */
	.bss ALIGN(8):
	{
		_bss_start = .;
		*boot_handlers.O(.bss .bss.* .scommon .sbss .dynbss COMMON)
		*(.bss .bss.*)
		*(.scommon)
		*(.sbss)
		*(.dynbss)
		*(COMMON)
		/* Align here to ensure that the .bss section occupies space up to
		_end.  Align after .bss to ensure correct alignment even if the
		.bss section disappears because there are no input sections.  */
		. = ALIGN(32 / 8);
		_bss_end = .;
	} > ram						/* in RAM */

	. = ALIGN (8);
	_empty_ram = .;

/* This symbol defines end of code/data sections. Heap starts here. */
	PROVIDE(end    	  = .);
```


```c
#if configDYNAMIC_HEAP_SIZE
extern unsigned char _empty_ram;

#define HEAP_START_ADDRESS    (void*)&_empty_ram
#if CFG_SOC_NAME == SOC_BK7231N
#define HEAP_END_ADDRESS      (void*)(0x00400000 + 192 * 1024)
#else
#define HEAP_END_ADDRESS      (void*)(0x00400000 + 256 * 1024)
#endif

static void *prvHeapGetHeaderPointer(void)
{
	return (void *)HEAP_START_ADDRESS;
}

static uint32_t prvHeapGetTotalSize(void)
{
	ASSERT(HEAP_END_ADDRESS > HEAP_START_ADDRESS);
	return (HEAP_END_ADDRESS - HEAP_START_ADDRESS);
}
```

想增加heap的大小，有两个思路：

1. 裁剪代码，减少 .data .bss 代码的大小
2. tcm 也是一类 ram，将部分代码放 tcm 区域，heap 所在 ram 这边就腾出空间来了。



codesize.py 脚本可以分析 map 文件，统计各个 lib、obj、section 占用的 ram 、flash 大小，分析结果输出为 csv 文件。

对csv文件按 .bss 或者 .data 的大小进行排序，看哪个地方占用的内存大，那么就可以查看这部分代码是否可以优化。


```python
#!/usr/bin/env python3
# Copyright (C) 2018 RDA Technologies Limited and/or its affiliates("RDA").
# All rights reserved.
#
# This software is supplied "AS IS" without any warranties.
# RDA assumes no responsibility or liability for the use of the software,
# conveys no license or title under any patent, copyright, or mask work
# right to the product. RDA reserves the right to make changes in the
# software without notification.  RDA also make no representation or
# warranty that such application will be suitable for the specified use
# without further testing or modification.

import os
import sys
import re
import optparse
import fnmatch

# section types
section_text = [".text", ".text.*", ".ram", "RESET", "VECTORS",
                ".sramboottext", '.sramtext', '.ramtext', ".boot_sector_start",
                ".boottext", ".irqtext", ".romtext", ".bootsramtext",
                ".ram", ".start_entry", ".exception", ".tlb_exception",
                ".tlbtext", ".syssram_L1_text", ".nbsram_patch_text",
                ".init", ".sramTEXT",".noinit"]
section_ro = [".rodata", ".rodata.*",
              ".bootrodata", ".roresdata", ".robsdata", ".extra"]
section_rw = [".data", ".data.*",
              ".rwkeep", ".bootsramdata", ".sramdata", ".sramucdata",
              ".srroucdata", ".ucdata"]
section_zi = [".bss", ".bss.*", "COMMON", ".scommon", ".sdata", ".sdata.*",
              ".sbss", ".sbss.*", ".nbsram_globals",
              ".sramuninit", ".sramucuninit", ".dsp_iq_data", ".ramucbss",
              ".backup", ".bootsrambss", ".ucbss", ".srambss", ".sramucbss",
              ".ucbackup", ".xcv_reg_value", ".abb_reg_value", ".pmu_reg_value",
              ".hal_boot_sector_reload_struct", ".boot_sector_reload_struct_ptr",
              ".boot_sector_struct", ".boot_sector_struct_ptr",
              ".fixptr", ".dbgfunc", ".sram_overlay", ".TTBL1", ".TTBL2"]
section_ignore = [".ARM.exidx", ".ARM.attributes", ".comment", ".debug_*",
                  ".iplt", ".rel.iplt", ".igot.plt", '.reginfo', ".mdebug.*",
                  ".pdr", '.rel.dyn','.noinit']


# check a name matches a list of patterns
def name_match(name, patterns):
    return any(fnmatch.fnmatch(name, pattern) for pattern in patterns)


# section type by section name
def section_type(section_name):
    if name_match(section_name, section_ignore):
        stype = "ignore"
    elif name_match(section_name, section_text):
        stype = "text"
    elif name_match(section_name, section_ro):
        stype = "rodata"
    elif name_match(section_name, section_rw):
        stype = "data"
    elif name_match(section_name, section_zi):
        stype = "bss"
    else:
        stype = "ignore"
#        print("unknown section:", section_name)
#        sys.exit(1)
    return stype


# make an entry, a section in one object file
def make_entry(section_name, size, obj):
    stype = section_type(section_name)
    r = re.compile(r'(\S+)\((\S+)\)')
    m = r.fullmatch(obj)
    fobj = obj
    if m:
        lib = m.group(1)
        obj = m.group(2)
    else:
        lib = obj

    lib = lib.split("/")[-1]
    obj = obj.split("/")[-1]
    return {"section": section_name, "stype": stype, "size": size,
            "lib": lib, "obj": obj, 'fobj': fobj}


# parse map file
def parse_map(fname):
    fh = open(fname, 'r')

    # "SECTION ADDRESS SIZE OBJ"
    r2 = re.compile(
        r"\s+(\S+)\s+0x([0-9a-fA-F]{8,16})\s+0x([0-9a-fA-F]+)\s+(\S+)")
    # "SECTION"
    r3 = re.compile(r"\s+(\S+)")
    # "ADDRESS SIZE OBJ"
    r4 = re.compile(r"\s+0x([0-9a-fA-F]{8,16})\s+0x([0-9a-fA-F]+)\s+(\S+)")

    ents = []
    map_found = False
    section_name = None
    for line in fh.readlines():
        if line.startswith("Linker script and memory map"):
            map_found = True
            continue
        if not map_found:
            continue
        if line.startswith('OUTPUT('):
            break

        line = line.rstrip()

        if section_name:
            m = r4.fullmatch(line)
            if m:
                size = int(m.group(2), 16)
                obj = m.group(3)
                ents.append(make_entry(section_name, size, obj))
            section_name = None
            continue

        m = r2.fullmatch(line)
        if m:
            name = m.group(1)
            size = int(m.group(3), 16)
            obj = m.group(4)
            ents.append(make_entry(name, size, obj))
            continue

        m = r3.fullmatch(line)
        if m:
            section_name = m.group(1)
            continue
    return ents


def sizelist(f):
    sl = [f["text"],
          f["rodata"],
          f["data"],
          f["bss"],
          f["text"] + f["rodata"],
          f["text"] + f["rodata"] + f["data"],
          f["data"] + f["bss"]]
    ss = [str(s) for s in sl]
    return ",".join(ss)


def main(argv):
    opt = optparse.OptionParser(usage="""usage %prog [options]

This utility will analyze code size from map file. Code size are expressed
in concept of: text, rodata, data, bss. There is a map inside this script
to map each section name to section type. When there is an unknown section
name, it is needed to add the section name to this script.
""")

    opt.add_option("--map", action="store", dest="map",
                   help="map file created at linking.")
    opt.add_option("--outobj", action="store", dest="outobj", default="outobj.csv",
                   help="detailed information by object file (default: outobj.csv)")
    opt.add_option("--outlib", action="store", dest="outlib", default="outlib.csv",
                   help="detailed information by library (default: outlib.csv)")
    opt.add_option("--outsect", action="store", dest="outsect", default="outsect.csv",
                   help="detailed information by section (default: outsect.csv)")

    opt, argv = opt.parse_args(argv)

    if not opt.map:
        print("No map file specified!")
        return 1

    ents = parse_map(opt.map)

    stype_total = {"text": 0, "rodata": 0,
                   "data": 0, "bss": 0, 'ignore': 0}
    for ent in ents:
        stype = ent['stype']
        size = ent['size']
        stype_total[stype] += size

    print("total: text,rodata,data,bss,code,flash,ram")
    print("      ", sizelist(stype_total))

    lib_total = {}
    for ent in ents:
        lib = ent['lib']
        stype = ent['stype']
        size = ent['size']
        if lib not in lib_total:
            lib_total[lib] = {"text": 0, "rodata": 0,
                              "data": 0, "bss": 0, 'ignore': 0}
        lib_total[lib][stype] += size

    print("size by library to %s ..." % (opt.outlib))
    fh = open(opt.outlib, "w")
    fh.write("library name,text,rodata,data,bss,code,flash,ram\n")
    for n, l in lib_total.items():
        fh.write("%s,%s\n" % (n, sizelist(l)))
    fh.close()

    obj_total = {}
    for ent in ents:
        fobj = ent['fobj']
        lib = ent['lib']
        obj = ent['obj']
        stype = ent['stype']
        size = ent['size']
        if fobj not in obj_total:
            obj_total[fobj] = {"lib": lib, "obj": obj, "text": 0, "rodata": 0,
                               "data": 0, "bss": 0, 'ignore': 0}
        obj_total[fobj][stype] += size

    print("size by object to %s ..." % (opt.outobj))
    fh = open(opt.outobj, "w")
    fh.write("object name,library name,text,rodata,data,bss,code,flash,ram\n")
    for n, l in obj_total.items():
        fh.write("%s,%s,%s\n" % (l['obj'], l["lib"], sizelist(l)))
    fh.close()

    section_total = {}
    for ent in ents:
        section = ent['section']
        stype = ent['stype']
        size = ent['size']
        if section not in section_total:
            section_total[section] = {"type": stype, "size": 0}
        section_total[section]['size'] += size

    print("size by section to %s ..." % (opt.outsect))
    fh = open(opt.outsect, "w")
    fh.write("section name,section type,size\n")
    for n, l in section_total.items():
        fh.write("%s,%s,%s\n" % (n, l['type'], l['size']))
    fh.close()

    return 0


if __name__ == '__main__':
    sys.exit(main(sys.argv[1:]))

```

执行脚本解析map文件：

```sh
~\Desktop via  v2.7.18
❯ py -3 .\codesize.py --map .\fc41d_bsp_app.map
total: text,rodata,data,bss,code,flash,ram
       979352,144557,4697,101218,1123909,1128606,105915
size by library to outlib.csv ...
size by object to outobj.csv ...
size by section to outsect.csv ...

```


![](https://github.com/hacperme/picx-images-hosting/raw/master/20240711/image.7ljvl4f6xi.webp)



tcm 使用了 50% 左右，lwip 内存管理相关的 mem memp 内存较大，可以将其放到到 tcm，从而减少 ram 的使用。

修改ld文件，在 tcm 内存段增加 mem memp 的配置。

```ld
*mem.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)
*memp.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)

*coap_net.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON) 
*coap_pdu.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)
*coap_uri.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)
```



另外 mbedtls 的 MBEDTLS_AES_ROM_TABLES 配置也可以优化，定义 MBEDTLS_AES_ROM_TABLES 可以将一些预定义的表放到rom，从而减少ram的使用。之前为了减少 rom 的空间使用关闭了 MBEDTLS_AES_ROM_TABLES。

```c
#define MBEDTLS_AES_ROM_TABLES
```

优化之后的内存使用情况：

```bash
Memory region         Used Size  Region Size  %age Used
           flash:     1138056 B      3912 KB     28.41%
             tcm:       60036 B        60 KB     97.71%
            itcm:        3848 B         4 KB     93.95%
             ram:       46196 B     196352 B     23.53%
```

heap 空间最终增加了接近 40kB。

总之，--print-memory-usage 看内存整体的使用情况，codesize.py 解析 map 文件查看内存使用细节，根据这些信息调整内存布局或者优化、裁剪代码。
