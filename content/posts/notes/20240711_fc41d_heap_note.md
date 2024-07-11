---
title: "FC41D heap 空间优化方法"
date: 2024-07-11T02:00:18+08:00
lastmod: 2024-07-11T02:00:18+08:00
author: ["hacper"]
tags:
   - FC41D
   - heap
   - gcc
   - freertos
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
~\Desktop via 🐍 v2.7.18
❯ py -3 .\codesize.py --map .\fc41d_bsp_app.map
total: text,rodata,data,bss,code,flash,ram
       979352,144557,4697,101218,1123909,1128606,105915
size by library to outlib.csv ...
size by object to outobj.csv ...
size by section to outsect.csv ...

```



![](https://jsd.cdn.zzko.cn/gh/hacperme/picx-images-hosting@master/20240711/image.7ljvl4f6xi.webp)



tcm 使用了50%左右，lwip 内存管理相关的 mem memp 内存较大，可以将其放到到 tcm，减少ram的使用。

修改ld文件，在tcm内存段增加 mem memp的配置。

```ld
*mem.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)
*memp.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)

*coap_net.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON) 
*coap_pdu.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)
*coap_uri.c.o(.bss .bss.* .scommon .sbss .dynbss COMMON)
```



mbedtls 的配置，定义 MBEDTLS_AES_ROM_TABLES 可以将一些预定义的表放到rom，从而减少ram的使用。

```c
#define MBEDTLS_AES_ROM_TABLES
```

优化之后的内存使用情况

```bash
Memory region         Used Size  Region Size  %age Used
           flash:     1138056 B      3912 KB     28.41%
             tcm:       60036 B        60 KB     97.71%
            itcm:        3848 B         4 KB     93.95%
             ram:       46196 B     196352 B     23.53%
```



总之，--print-memory-usage 看内存使用情况，codesize.py 查看内存使用细节，根据这些信息调整内存布局或者优化代码。