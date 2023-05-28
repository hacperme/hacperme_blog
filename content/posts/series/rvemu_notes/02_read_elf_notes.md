---
title: "rvemu 学习笔记-读取 ELF 文件"
date: 2023-05-21T23:15:59+08:00
lastmod: 2023-05-21T23:15:59+08:00
author: ["hacper"]
tags:
    - rvemu
    - risc-v
    - ELF
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

## 视频

{{< bilibili BV1jj411c7dJ >}}



## ELF 文件格式

### 类型

ELF 格式可以表示 4 种类型的二进制文件。

- 可重定位文件(relocatable file)，包含由编译器生成的代码以及数据。链接器会将它与其它目标文件链接起来从而创建可执行文件或者共享目标文件。在 Linux 系统中，这种文件的后缀一般为 `.o` 。

- 可执行文件(executable file)

- 共享库文件(shared object file)，.so文件

- 核心转储文件(core dump file)

### ELF 文件内容布局

ELF 文件有两种视图，链接视图和执行视图。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.537j262ygd80.webp)

- ELF Header, ELF头部，定义全局性信息
- Program Header Table， 描述段(Segment)信息的数组，每个元素对应一个段；通常包含在可执行文件中，可重定文件中可选(通常不包含)
- Segment and Section，段(Segment)由若干区(Section)组成；段在运行时被加载到进程地址空间中，包含在可执行文件中；区是段的组成单元，包含在可执行文件和可重定位文件中
- Section Header Table，描述区(Section)信息的数组，每个元素对应一个区；通常包含在可重定位文件中，可执行文件中为可选(通常包含)



![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/Elf-layout--en.57iifa9anbs0.svg)

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.7g9d6hz09800.webp)

### ELF Header 数据结构

ELF Header中各字段的定义如下。

```c
#define EI_NIDENT 16

typedef struct {
    u8 e_ident[EI_NIDENT];
    u16 e_type;
    u16 e_machine;
    u32 e_version;
    u64 e_entry;
    u64 e_phoff;
    u64 e_shoff;
    u32 e_flags;
    u16 e_ehsize;
    u16 e_phentsize;
    u16 e_phnum;
    u16 e_shentsize;
    u16 e_shnum;
    u16 e_shstrndx;
} elf64_ehdr_t;
```

注意 32 位和 64 位的 ELF Header 有点不同（字段一致，表示内存地址的大小由4字节和8字节的区别而已）。

e_ident 包含16 字节，包含以下字段

| 宏名称        | 下标 | 目的                |
| :------------ | :--- | :------------------ |
| EI_MAG0       | 0    | 文件标识            |
| EI_MAG1       | 1    | 文件标识            |
| EI_MAG2       | 2    | 文件标识            |
| EI_MAG3       | 3    | 文件标识            |
| EI_CLASS      | 4    | 文件类              |
| EI_DATA       | 5    | 数据编码            |
| EI_VERSION    | 6    | 文件版本            |
| EI_OSABI      | 7    | 目标操作系统ABI标识 |
| EI_ABIVERSION | 8    | ABI 版本            |
| EI_PAD        | 9    | 补齐字节开始处      |

前 4 个字节 EI_MAG0 ~ EI_MAG3 为 ELF 文件的标识，固定值：0x7f 0x45 0x4c 0x46。

EI_CLASS 文件类，这个字节被设置成1或者2来表示32位或者64位的格式。

EI_DATA 数据编码，这个字节被设置为1或2来表示小端序或大端序。

EI_VERSION 文件版本，固定值 1.

EI_OSABI 目标操作系统ABI标识：

| Value |                             ABI                              |
| :---: | :----------------------------------------------------------: |
| 0x00  |      [System V](https://en.wikipedia.org/wiki/System_V)      |
| 0x01  |         [HP-UX](https://en.wikipedia.org/wiki/HP-UX)         |
| 0x02  |        [NetBSD](https://en.wikipedia.org/wiki/NetBSD)        |
| 0x03  |         [Linux](https://en.wikipedia.org/wiki/Linux)         |
| 0x04  |      [GNU Hurd](https://en.wikipedia.org/wiki/GNU_Hurd)      |
| 0x06  | [Solaris](https://en.wikipedia.org/wiki/Solaris_(operating_system)) |
| 0x07  |   [AIX](https://en.wikipedia.org/wiki/IBM_AIX) (Monterey)    |
| 0x08  |          [IRIX](https://en.wikipedia.org/wiki/IRIX)          |
| 0x09  |       [FreeBSD](https://en.wikipedia.org/wiki/FreeBSD)       |
| 0x0A  |         [Tru64](https://en.wikipedia.org/wiki/Tru64)         |
| 0x0B  |                        Novell Modesto                        |
| 0x0C  |       [OpenBSD](https://en.wikipedia.org/wiki/OpenBSD)       |
| 0x0D  |       [OpenVMS](https://en.wikipedia.org/wiki/OpenVMS)       |
| 0x0E  | [NonStop Kernel](https://en.wikipedia.org/wiki/NonStop_(server_computers)) |
| 0x0F  | [AROS](https://en.wikipedia.org/wiki/AROS_Research_Operating_System) |
| 0x10  |                           FenixOS                            |
| 0x11  |   Nuxi [CloudABI](https://en.wikipedia.org/wiki/CloudABI)    |
| 0x12  | [Stratus Technologies OpenVOS](https://en.wikipedia.org/wiki/Stratus_VOS) |

EI_ABIVERSION 指定ABI 版本。

EI_PAD 表示其后是填充数据，当前未使用字段，值为 0。



e_type 标识目标文件类型。

| Value  |   Type    |                       Meaning                        |
| :----: | :-------: | :--------------------------------------------------: |
|  0x00  |  ET_NONE  |                       Unknown.                       |
|  0x01  |  ET_REL   |                  Relocatable file.                   |
|  0x02  |  ET_EXEC  |                   Executable file.                   |
|  0x03  |  ET_DYN   |                    Shared object.                    |
|  0x04  |  ET_CORE  |                      Core file.                      |
| 0xFE00 |  ET_LOOS  | Reserved inclusive range. Operating system specific. |
| 0xFEFF |  ET_HIOS  | Reserved inclusive range. Operating system specific. |
| 0xFF00 | ET_LOPROC |    Reserved inclusive range. Processor specific.     |
| 0xFFFF | ET_HIPROC |    Reserved inclusive range. Processor specific.     |

e_machine 指定目标指令集体系结构。

|    Value    |                             ISA                              |
| :---------: | :----------------------------------------------------------: |
|    0x00     |                 No specific instruction set                  |
|    0x01     |  [AT&T WE 32100](https://en.wikipedia.org/wiki/Bellmac_32)   |
|    0x02     |         [SPARC](https://en.wikipedia.org/wiki/SPARC)         |
|    0x03     |           [x86](https://en.wikipedia.org/wiki/X86)           |
|    0x04     | [Motorola 68000 (M68k)](https://en.wikipedia.org/wiki/Motorola_68000_series) |
|    0x05     | [Motorola 88000 (M88k)](https://en.wikipedia.org/wiki/Motorola_88000) |
|    0x06     | [Intel MCU](https://en.wikipedia.org/wiki/List_of_Intel_processors#Microcontrollers) |
|    0x07     |   [Intel 80860](https://en.wikipedia.org/wiki/Intel_i860)    |
|    0x08     |   [MIPS](https://en.wikipedia.org/wiki/MIPS_architecture)    |
|    0x09     | [IBM System/370](https://en.wikipedia.org/wiki/IBM_System/370) |
|    0x0A     | [MIPS RS3000 Little-endian](https://en.wikipedia.org/wiki/R3000) |
| 0x0B - 0x0D |                   Reserved for future use                    |
|    0x0E     | [Hewlett-Packard PA-RISC](https://en.wikipedia.org/wiki/PA-RISC) |
|    0x0F     |                   Reserved for future use                    |
|    0x13     |   [Intel 80960](https://en.wikipedia.org/wiki/Intel_i960)    |
|    0x14     |       [PowerPC](https://en.wikipedia.org/wiki/PowerPC)       |
|    0x15     |  [PowerPC](https://en.wikipedia.org/wiki/PowerPC) (64-bit)   |
|    0x16     | [S390](https://en.wikipedia.org/wiki/Z/Architecture), including S390x |
|    0x17     |                         IBM SPU/SPC                          |
| 0x18 - 0x23 |                   Reserved for future use                    |
|    0x24     |        [NEC V800](https://en.wikipedia.org/wiki/V850)        |
|    0x25     |                         Fujitsu FR20                         |
|    0x26     |       [TRW RH-32](https://en.wikipedia.org/wiki/RH-32)       |
|    0x27     |                         Motorola RCE                         |
|    0x28     | [Arm](https://en.wikipedia.org/wiki/ARM_architecture_family) (up to Armv7/AArch32) |
|    0x29     | [Digital Alpha](https://en.wikipedia.org/wiki/Digital_Alpha) |
|    0x2A     |        [SuperH](https://en.wikipedia.org/wiki/SuperH)        |
|    0x2B     |    [SPARC Version 9](https://en.wikipedia.org/wiki/SPARC)    |
|    0x2C     | [Siemens TriCore embedded processor](https://en.wikipedia.org/wiki/Infineon_TriCore) |
|    0x2D     | [Argonaut RISC Core](https://en.wikipedia.org/wiki/ARC_(processor)) |
|    0x2E     |  [Hitachi H8/300](https://en.wikipedia.org/wiki/H8_Family)   |
|    0x2F     |  [Hitachi H8/300H](https://en.wikipedia.org/wiki/H8_Family)  |
|    0x30     |    [Hitachi H8S](https://en.wikipedia.org/wiki/H8_Family)    |
|    0x31     |  [Hitachi H8/500](https://en.wikipedia.org/wiki/H8_Family)   |
|    0x32     |         [IA-64](https://en.wikipedia.org/wiki/IA-64)         |
|    0x33     |   [Stanford MIPS-X](https://en.wikipedia.org/wiki/MIPS-X)    |
|    0x34     | [Motorola ColdFire](https://en.wikipedia.org/wiki/NXP_ColdFire) |
|    0x35     | [Motorola M68HC12](https://en.wikipedia.org/wiki/Motorola_68HC12) |
|    0x36     |              Fujitsu MMA Multimedia Accelerator              |
|    0x37     |                         Siemens PCP                          |
|    0x38     | [Sony nCPU embedded RISC processor](https://en.wikipedia.org/wiki/Cell_(microprocessor)) |
|    0x39     |                  Denso NDR1 microprocessor                   |
|    0x3A     |                 Motorola Star*Core processor                 |
|    0x3B     |                    Toyota ME16 processor                     |
|    0x3C     |              STMicroelectronics ST100 processor              |
|    0x3D     |     Advanced Logic Corp. TinyJ embedded processor family     |
|    0x3E     |      [AMD x86-64](https://en.wikipedia.org/wiki/Amd64)       |
|    0x3F     |                      Sony DSP Processor                      |
|    0x40     | [Digital Equipment Corp. PDP-10](https://en.wikipedia.org/wiki/PDP-10) |
|    0x41     | [Digital Equipment Corp. PDP-11](https://en.wikipedia.org/wiki/PDP-11) |
|    0x42     |                 Siemens FX66 microcontroller                 |
|    0x43     |       STMicroelectronics ST9+ 8/16 bit microcontroller       |
|    0x44     |         STMicroelectronics ST7 8-bit microcontroller         |
|    0x45     | [Motorola MC68HC16 Microcontroller](https://en.wikipedia.org/wiki/Motorola_68HC16) |
|    0x46     | [Motorola MC68HC11 Microcontroller](https://en.wikipedia.org/wiki/Motorola_68HC11) |
|    0x47     | [Motorola MC68HC08 Microcontroller](https://en.wikipedia.org/wiki/Motorola_68HC08) |
|    0x48     | [Motorola MC68HC05 Microcontroller](https://en.wikipedia.org/wiki/Motorola_68HC05) |
|    0x49     |                     Silicon Graphics SVx                     |
|    0x4A     |        STMicroelectronics ST19 8-bit microcontroller         |
|    0x4B     |       [Digital VAX](https://en.wikipedia.org/wiki/VAX)       |
|    0x4C     |        Axis Communications 32-bit embedded processor         |
|    0x4D     |       Infineon Technologies 32-bit embedded processor        |
|    0x4E     |               Element 14 64-bit DSP Processor                |
|    0x4F     |                LSI Logic 16-bit DSP Processor                |
|    0x8C     | [TMS320C6000 Family](https://en.wikipedia.org/wiki/Texas_Instruments_TMS320#C6000_series) |
|    0xAF     | [MCST Elbrus e2k](https://en.wikipedia.org/wiki/Elbrus_2000) |
|    0xB7     | [Arm 64-bits](https://en.wikipedia.org/wiki/AArch64) (Armv8/AArch64) |
|    0xDC     |     [Zilog Z80](https://en.wikipedia.org/wiki/Zilog_Z80)     |
|    0xF3     |        [RISC-V](https://en.wikipedia.org/wiki/RISC-V)        |
|    0xF7     | [Berkeley Packet Filter](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter) |
|    0x101    |    [WDC 65C816](https://en.wikipedia.org/wiki/WDC_65C816)    |

e_version 标识目标文件的版本，固定值 1。

e_entry 程序入口函数地址。此字段长度为32位或64位，具体取决于前面定义的格式（字节0x04）。如果文件没有关联的入口点，则其值为零。

e_phoff 程序头表的开始位置。

e_shoff  节头表在文件中的字节偏移。

e_flags 表示与CPU处理器架构相关的信息。

e_ehsize  ELF 文件头部的字节长度。

e_phentsize 程序头部表中每个表项的字节长度。

e_phnum 表示program header table中元素个数

e_shentsize 表示section header table中每个元素的大小

e_shnum 表示section header table中元素的个数

e_shstrndx 表示描述各section字符名称的string table在section header table中的下标

### ELF Program Header Table 数据结构

Program Header Table 是一个结构体数组，数据大小也有32位和64位程序之分。

ELF 头中的 `e_phentsize` 和 `e_phnum` 指定了该数组每个元素的大小以及元素个数。一个目标文件的段包含一个或者多个节。**程序的头部只有对于可执行文件和共享目标文件有意义。**

可以说，Program Header Table 就是专门为 ELF 文件运行时中的段所准备的。

```c
typedef struct {
    u32 p_type;
    u32 p_flags;
    u64 p_offset;
    u64 p_vaddr;
    u64 p_paddr;
    u64 p_filesz;
    u64 p_memsz;
    u64 p_align;
} elf64_phdr_t;
```

p_type 段的类型。

|   Value    |    Name    |                           Meaning                            |
| :--------: | :--------: | :----------------------------------------------------------: |
| 0x00000000 |  PT_NULL   |              Program header table entry unused.              |
| 0x00000001 |  PT_LOAD   | Loadable segment.此类型header元素描述了可加载到进程空间的代码段或数据段 |
| 0x00000002 | PT_DYNAMIC | Dynamic linking information.此类型header元素描述了动态加载段，其内部通常包含了一个名为”.dynamic”的动态加载区；这也是一个数组，每个元素描述了与动态加载相关的各方面信息 |
| 0x00000003 | PT_INTERP  | Interpreter information.该段内存记录了动态加载解析器的访问路径字符串 |
| 0x00000004 |  PT_NOTE   |                    Auxiliary information.                    |
| 0x00000005 |  PT_SHLIB  |                          Reserved.                           |
| 0x00000006 |  PT_PHDR   |     此类型header元素描述了program header table自身的信息     |
| 0x00000007 |   PT_TLS   |                Thread-Local Storage template.                |
| 0x60000000 |  PT_LOOS   |     Reserved inclusive range. Operating system specific.     |
| 0x6FFFFFFF |  PT_HIOS   |     Reserved inclusive range. Operating system specific.     |
| 0x70000000 | PT_LOPROC  |        Reserved inclusive range. Processor specific.         |
| 0x7FFFFFFF | PT_HIPROC  |        Reserved inclusive range. Processor specific.         |

p_flags 段权限。

![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.1h9i5z63s128.webp)

p_offset 从文件开始到该段开头的第一个字节的偏移。

p_vaddr 该段第一个字节在内存中的虚拟地址。

p_paddr 在物理地址相关的系统中，保留给段的物理地址。

p_filesz 该字段给出了文件镜像中该段的大小。

p_memsz 该字段给出了内存镜像中该段的大小。

p_align 可加载的程序的段的 p_vaddr 以及 p_offset 的大小必须是 page 的整数倍。该成员给出了段在文件以及内存中的对齐方式。如果该值为 0 或 1 的话，表示不需要对齐。除此之外，p_align 应该是 2 的整数指数次方，并且 p_vaddr 与 p_offset 在模 p_align 的意义下，应该相等。

### 段(segment)类型

一个段 segment 可能包括一到多个节区 section。

- 代码段

  代码段只包含只读的指令以及数据

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.6e9i5f08h340.webp)

- 数据段

  数据段包含可写的数据以及以及指令

  ![](https://cdn.staticaly.com/gh/hacperme/picx_hosting@master/20210507/image.3y9bqzlxiig0.webp)

  .bss 节的类型为 SHT_NOBITS，这表明它在 ELF 文件中不占用空间，但是它却占用可执行文件的内存镜像的空间。通常情况下，没有被初始化的数据在段的尾部，因此，`p_memsz` 才会比 `p_filesz` 大。



### ELF Section Header Table 数据结构



```c
typedef struct {
	Elf64_Half	sh_name;	/* section name */
	Elf64_Half	sh_type;	/* section type */
	Elf64_Xword	sh_flags;	/* section flags */
	Elf64_Addr	sh_addr;	/* virtual address */
	Elf64_Off	sh_offset;	/* file offset */
	Elf64_Xword	sh_size;	/* section size */
	Elf64_Half	sh_link;	/* link to another */
	Elf64_Half	sh_info;	/* misc info */
	Elf64_Xword	sh_addralign;	/* memory alignment */
	Elf64_Xword	sh_entsize;	/* table entry size */
} Elf64_Shdr;
```



### Symbol Table



```c
typedef struct {
	Elf64_Half	st_name;	/* Symbol name index in str table */
	Elf_Byte	st_info;	/* type / binding attrs */
	Elf_Byte	st_other;	/* unused */
	Elf64_Quarter	st_shndx;	/* section index of symbol */
	Elf64_Xword	st_value;	/* value of symbol */
	Elf64_Xword	st_size;	/* size of symbol */
} Elf64_Sym;
```



## readelf 工具使用



## 代码实现



## mmap 的使用









## 参考资料

- [Executable and Linkable Format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
- [ELF文件格式解析](https://www.openeuler.org/zh/blog/lijiajie128/2020-11-03-ELF%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E8%A7%A3%E6%9E%90.html) https://www.openeuler.org/zh/blog/lijiajie128/2020-11-03-ELF%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E8%A7%A3%E6%9E%90.html
- [ELF 文件](https://ctf-wiki.org/executable/elf/structure/basic-info/) https://ctf-wiki.org/executable/elf/structure/basic-info/