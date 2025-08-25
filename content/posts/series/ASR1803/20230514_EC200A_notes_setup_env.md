---
title: "【EC200A 学习笔记】-- 搭建开发环境"
date: 2023-05-14T00:49:37+08:00
lastmod: 2023-05-14T00:49:37+08:00
author: ["hacper"]
tags:
    - ASR1803
    - EC200A
    - OpenWrt
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

## 软件安装包和环境概览

- VMware
    虚拟机，运行 ubuntu 系统，用于源码编译。
    
- Quectel_Windows_USB_Driver(A)_Customer_V1.0.9.zip
    usb 驱动，烧写固件查看日志的前提。
    
- CATStudio
    日志工具
    
- MobaXterm

    非必须，看串口日志，登录系统。

- adb

    传输文件，登录shell等。

- SWDownloader_4_9_0_4.7z

    烧写固件工具

## 步骤

### 安装软件依赖

安装 ubuntu 2202.4 虚拟机环境，尝试过 wsl2 ， 编译不行，还是得上 VMware。

ubuntu  需要安装软件包：

```bash
sudo apt install build-essential bison flex zlib1g-dev libncurses5-dev subversion quilt intltool ruby fastjar zip unzip gawk git-core python-is-python3 python3 -y
```

### 获取代码 

配置 git：

```bash
abc@abc-virtual-machine:~$ git config --global user.name abc.abc
abc@abc-virtual-machine:~$ git config --global user.email abc.abc@abc.com
```

配置ssh：

密钥已经有了，且添加公钥到了 gitlab 服务器上，这里只拷贝添加一下私钥，用于拉代码。

```bash
cd ~
mkdir .ssh
cd .ssh/
mv ~/Desktop/id_rsa_p_id_ed25519 .
sudo chmod 400 id_rsa_p_id_ed25519 
ssh-add id_rsa_p_id_ed25519
ssh-agent 
```

拉取代码：

```bash
git clone ssh://git@git-master.quectel.com:8407/ec200a/ec200a_linux.git -b master_r02
```

### 编译

编译命令：

```bash
# 配置环境变量
source package/quectel/compile/ql_build_config
# 查看可选配置
buildconfig
# 配置编译的目标产品
buildconfig EC200A_ELAA EC200A_ELAA_v01 STD

# 执行编译
build_fw
```

### 烧录

烧写固件之前需要安装 usb 驱动，解压 Quectel_Windows_USB_Driver(A)_Customer_V1.0.9.zip 压缩包进行安装即可。

生成固件在路径：```bin/target```

```bash
abc@abc-virtual-machine:~/ec200a_linux$ tree bin/target/
bin/target/
├── EC20001A01M1G
│   ├── dbg
│   │   ├── Boerne_DIAG.mdb.txt
│   │   ├── CRANE_DS_XIP_DM_GENERIC_DIAG.mdb
│   │   ├── CRANE_DS_XIP_DM_GENERIC_NVM.mdb
│   │   └── MDB.txt
│   ├── mversion
│   └── update
│       ├── ARBEL.bin
│       ├── MSA.bin
│       ├── oem_data.ubi
│       ├── quectel_skylark_pm802_standard_AB_1G.blf
│       ├── RFPLUGIN.bin
│       ├── root.squashfs
│       ├── TLoader_QSPINAND.bin
│       ├── u-boot.bin
│       └── zImage
├── EC20001A01M1G_fota
│   ├── md5sums
│   ├── mversion
│   └── update
│       ├── quectel_skylark_pm802_AB_OTA_local.img
│       ├── quectel_skylark_pm802_AB_OTA_url.img
│       └── quectel_skylark_pm802_standard_AB_1G.blf
└── EC20001A01M1G.zip

5 directories, 20 files

```

烧录使用 SWDownloader 工具打开 quectel_skylark_pm802_standard_AB_1G.blf，先点击左上角绿色的指示灯按钮，复位模组即可进行固件烧写。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230426140707714.5zgpr7hxqyw0.webp)

