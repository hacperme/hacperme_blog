---
title: "MAC 绑定 License 的通用解决方案（Docker/WSL 双方案）"
date: 2026-08-05T00:00:00+08:00
lastmod: 2026-08-05T00:00:00+08:00
author: ["hacper"]
tags:
    - license
    - flexlm
    - docker
    - wsl
    - mac地址
    - 工具链
categories:
    - 笔记
description: "商用工具链 license 绑定 MAC 地址时的通用解法：Docker 指定 MAC 或 WSL 新增 bond 网卡"
summary: "FlexLM/FlexNet 类 license 绑定 MAC 时，用 Docker --mac-address 或 WSL bond 网卡指定 MAC 即可绕过主机绑定，通用且不污染宿主机网络。"
slug: "mac-license-solution"
draft: true
comments: true
showToc: true
TocOpen: true
autonumbering: true
hidemeta: false
disableShare: true
searchHidden: false
showbreadcrumbs: true
---

# MAC 绑定 License 的通用解决方案

## 背景

很多商用工具链（Cadence Xtensa XCC、IAR、部分 Keil 版本等）使用 **FlexLM/FlexNet** 做许可证管理，license 文件通过 `HOSTID` 字段**绑定主机 MAC 地址**：

```text
INCREMENT XT_XCC_TIE_xxxxxx xtensad 15.0 30-dec-2026 uncounted \
    xxxxxxxx HOSTID=345a603923bb SN=xxx TS_OK \
    SIGN="xxxx xxxx xxxx ..."
```

`HOSTID=345a603923bb` 即绑定 MAC `34:5a:60:39:23:bb`。换了机器（MAC 不匹配）就无法编译，找厂商重发 license 又费时费力。

## 思路

FlexLM 校验的是**系统网卡上是否存在 license 中绑定的 MAC 地址**。所以核心解法就是：**让当前系统存在一个该 MAC 地址的网卡**。两个不污染物理网络的方式：

## 方案一：Docker 容器指定 MAC（推荐）

Docker 创建容器时可以指定网卡 MAC 地址，容器内跑编译即可命中 license：

```bash
docker run --rm -it \
  --hostname my-dev-host \
  -v $(pwd):/project \
  -v /opt/:/opt/ \
  -v /opt/license/:/license/ \
  --mac-address 34:5a:60:35:14:5d \
  --env-file docker.env \
  your-registry/toolchain_ubuntu20.04:latest \
  /bin/bash -c 'cd /project/sdk/ && ./build.sh -c && ./build.sh -all'
```

要点：
- `--mac-address`：指定容器内网卡的 MAC（必须与 license HOSTID 对应）
- `-v /opt/license/:/license/`：挂载 license 文件目录
- `-v $(pwd):/project`：挂载工程
- `--rm`：编译完自动删除，宿主机网络零污染

> 适合有 Docker 的环境，一次性镜像，可复现、可交付。

## 方案二：WSL 新增 bond 网卡指定 MAC

WSL2 环境（无 Docker）可以用 `ip link` 创建一个虚拟 bond 网卡并直接设置 MAC：

```bash
sudo ip link add bond0 type bond
sudo ip link set bond0 address 34:5a:60:39:23:bb
./build.sh -all
```

- `bond0` 是虚拟网卡，不影响物理网卡
- 设置 MAC 后系统就能被 license 校验命中
- 注意：WSL 重启后 bond0 会消失，需重新创建（可写进 `.bashrc` 或启动脚本）

## 通用性说明

这个方法适用于**所有 FlexLM/FlexNet 类 MAC 绑定 license 的商用工具链**：

| 工具链 | 厂商 | license 类型 |
|--------|------|-------------|
| Xtensa XCC/Xplorer | Cadence | FlexLM（HOSTID=MAC） |
| IAR EW 系列 | IAR Systems | FlexLM（部分版本） |
| Keil MDK | ARM | FlexLM（部分浮点授权） |

### 注意事项
1. **只解决 MAC 绑定**：若 license 还校验 `HOSTID=ANY` 之外的字段（如网卡名、hostname），需配合 `--hostname` 等参数
2. **合法性**：请确保你拥有该 license 的使用权（厂商授权或官方 FAE 提供），本文仅分享技术方案
3. **有效期**：license 有有效期（如 30-dec-2026），过期需重新申请
4. Docker 方案建议把镜像、license、构建命令固化到项目文档，方便团队复用

## 小结

- **Docker `--mac-address`**：干净、可复现，推荐首选
- **WSL bond 网卡**：轻量、无 Docker 时兜底
- 本质都是"虚拟网卡指定 MAC 命中 FlexLM 校验"——一个思路，两个载体

---

*本文基于某 DSP 工具链（Cadence Xtensa XCC）license 的实战经验整理，方案本身通用。*
