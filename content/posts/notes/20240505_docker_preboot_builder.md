---
title: "使用 docker 构建 preboot 交叉编译环境"
date: 2024-05-05T02:00:18+08:00
lastmod: 2024-05-07T02:00:18+08:00
author: ["hacper"]
tags:
    - docker
    - linux
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

ASR1606/ASR1603 的 preboot 代码需要在 Linux 环境下编译，通常使用 VMware 或者 VirtualBox 软件创建一个 Linux 虚拟机，在虚拟机中做交叉编译。但 preboot 不是那种需要经常编译的代码，完全可以将 preboot 的编译环境制作成 docker 镜像，与虚拟机相比，docker 的启动速度更快，占用资源更少，需要用到的时候创建一个 docker 容器来编译 preboot，非常适合用来做这类偶尔需要用一下的场景。



## 构建镜像

docker 根据 dockerfile 文件来构建镜像，dockerfile 中的命令描述其实就是搭建交叉编译环境的过程，主要将工具链、编译脚本复制到镜像内并配置好环境变量。

```dockerfile
FROM ubuntu:latest
LABEL version="1.0.0"
LABEL author="hacper"

RUN apt update && apt install make unzip curl -y &&  mkdir /preboot && mkdir /preboot/src
COPY gcc-arm-none-eabi-9-2019-q4-major.zip /preboot
COPY build_CRANEL.sh /preboot
COPY build_CRANEM.sh /preboot
WORKDIR /preboot
RUN unzip gcc-arm-none-eabi-9-2019-q4-major.zip && rm gcc-arm-none-eabi-9-2019-q4-major.zip
ENV PATH=$PATH:/preboot/gcc-arm-none-eabi-9-2019-q4-major/bin
```

依赖的工具链 gcc-arm-none-eabi-9-2019-q4-major.zip，需要将其放在根目录。然后执行 docker build 命令构建镜像：

```bash
docker build -t hacper/preboot_builder:latest .
```

镜像构建完成之后可以将镜像推送到 [docker hub](https://hub.docker.com/)，后面使用的时候直接从  [docker hub](https://hub.docker.com/) 拉取构建好的镜像。

```bash
docker push hacper/preboot_builder
```

## 使用

后面使用只需要从  [docker hub](https://hub.docker.com/) 拉取镜像:

```bash
docker pull hacper/preboot_builder
```

然后创建容器：

```bash
docker run -v /e/workspace/preboot/preboot_boot2:/preboot/src -it hacper/preboot_builder
```

/e/workspace/preboot/preboot_boot2 只是示例路径，需要修改为自己电脑上的 preboot 源码路径。preboot_boot2 目录下的 CRANEL、CRANEM 分别放置对应的 preboot 源码，将 preboot_boot2 目录挂载到容器中。

执行编译脚本编译preboot

```bash
 ./build_CRANEL.sh
 ./build_CRANEM.sh
```

生成的镜像分别在路径：

```powershell
preboot_boot2\CRANEL\apps\preboot\bin\crane\preboot.bin

preboot_boot2\CRANEM\apps\preboot\bin\cortexr-arom-crane\preboot.bin
```

