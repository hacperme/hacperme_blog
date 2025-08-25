---
title: "EC200A 通过 CATsudio 导出系统文件方法"
date: 2023-05-12T00:16:46+08:00
lastmod: 2023-05-12T00:16:46+08:00
author: ["hacper"]
tags:
    - ASR1803
    - CATstudio
    - EC200A
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

ASR EC200A 平台可以通过CATsudio 的 Flashexplorer 导出模组文件系统内的文件。除了文件导出，Flashexplorer 也支持文件的增删查改。 

## 使用步骤

先要匹配 mdb 文件，EC200A 有两个mdb 文件， 一个是cp侧的（MDB.txt），另一个是ap侧的（Boerne_DIAG.mdb.txt）。

两个文件更新之后，右上角两个图标显示绿色，表示匹配正确。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230324134658891.5xq6v7uiod40.png)

然后在工具栏选择 Flashexplorer

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230324135129870.5u1r8n5beic0.png)

注意 EC200A 要选择 Application Core，与 cat 1 的不同之处。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230324134938806.5rxqq83vww80.png)

之后会显示文件目录树，在下面的窗口，可以对这些文件进行打开，修改、重命名等动作。

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230324134529099.5yb2ceehy140.png)



![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230324134604315.76fdytdh6go0.png)

![](https://github.com/hacperme/picx_hosting/raw/master/20210507/image-20230324134604315.76fdytdh6go0.png)

## 参考文档

- 《FlashExplorer_UserGuide_chs.pdf》