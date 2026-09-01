---
title: "WSL2 下搭建 ESP32-C3 Zephyr 开发环境"
date: 2026-09-01T00:00:00+08:00
lastmod: 2026-09-01T00:00:00+08:00
author: ["hacper"]
tags:
    - zephyr
    - esp32c3
    - wsl2
    - 嵌入式
    - rtos
    - 开发环境
categories:
    - 笔记
description: "在 WSL2 中从零搭建 ESP32-C3（XIAO ESP32C3）Zephyr 开发环境：环境依赖、USB 设备直通、编译、烧录与串口监视的完整流程。"
summary: "WSL2 + Zephyr + ESP32-C3 完整踩坑记录：apt 依赖、west 工具链、usbipd-win 共享 USB 到 WSL2、dialout 权限，以及 west build / flash / espressif monitor 三板斧。"
slug: "wsl2-esp32c3-zephyr"
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

# WSL2 下搭建 ESP32-C3 Zephyr 开发环境

## 背景

Zephyr 是一个面向资源受限设备的小型实时操作系统（RTOS），支持 500+ 开发板，包括乐鑫的 ESP32 系列。ESP32-C3 是乐鑫的 RISC-V 单核 Wi-Fi/BLE SoC（160MHz，4MB Flash），开发板用的是 Seeed XIAO ESP32C3（QFN32，自带 USB-Serial/JTAG，一根 Type-C 线就能烧录和看日志，非常方便）。

开发环境放在 WSL2 里，好处是宿主机 Windows 干净，工具链都隔离在发行版里，出问题重装即可。Zephyr 官方对 Ubuntu 支持最好，下面以 Ubuntu（WSL2）为例。

## 前置条件

- Windows 10/11，已安装 WSL2（`wsl --install`），发行版建议 Ubuntu 22.04/24.04
- 一个 ESP32-C3 开发板（本文用 Seeed XIAO ESP32C3）
- 管理员权限的 PowerShell（后面 usbipd 需要）

## 1. 安装环境依赖工具

参考官方指导：[Zephyr Getting Started](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)

```bash
sudo apt update
sudo apt upgrade

sudo apt install --no-install-recommends git cmake ninja-build gperf \
  ccache dfu-util device-tree-compiler wget python3-dev python3-venv python3-tk \
  xz-utils file make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1

# 验证版本
cmake --version
python3 --version
dtc --version
```

## 2. 安装 west 与 Zephyr 源码

```bash
python3 -m venv ~/zephyrproject/.venv
source ~/zephyrproject/.venv/bin/activate

pip install west

west init -m https://github.com/zephyrproject-rtos/zephyr ~/zephyrproject
cd ~/zephyrproject
west update

west packages pip --install
west zephyr-export
```

> `west update` 会拉取 Zephyr 主仓库和所有模块（hal、mcuboot 等），网络不好时容易卡，可以挂代理或换国内镜像加速。

## 3. 安装 Zephyr SDK

```bash
cd ~/zephyrproject/zephyr
west sdk install
```

## 4. 共享 USB 设备到 WSL2

WSL2 是虚拟机，默认访问不到宿主机 USB 设备，需要借助微软官方的 [usbipd-win](https://github.com/dorssel/usbipd-win) 做 USB/IP 直通。

### 4.1 安装 usbipd-win

```powershell
winget install usbipd
```

### 4.2 命令行绑定并连接

在**管理员 PowerShell** 中执行：

```powershell
# 列出 USB 设备，找到 ESP32-C3 对应的 BUSID
usbipd list

# 绑定（只需做一次，绑定后设备对 WSL 可见）
usbipd bind --busid=<BUSID>

# 连接到当前 WSL 发行版
usbipd attach --wsl --busid=<BUSID>
```

之后在 WSL 里用 `lsusb` 验证，能看到 `303a:1001`（Espressif USB JTAG/serial debug unit）就说明直通成功：

```text
$ lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 004: ID 303a:1001 Espressif USB JTAG/serial debug unit
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
```

### 4.3 图形界面方案：wsl-dashboard（可选）

不想敲命令行的话，可以装 [wsl-dashboard](https://github.com/owu/wsl-dashboard)，在它的 UI 里直接绑定和连接 USB 设备，适合偶尔用一下的场景。

### 4.4 串口权限（dialout 组）

设备直通后，`/dev/ttyACM0` 默认属于 `dialout` 组，普通用户没有读写权限：

```text
$ ls -l /dev/ttyACM*
crw-rw---- 1 root dialout 166, 0 Sep  1 16:15 /dev/ttyACM0
```

把自己加进 `dialout` 组，然后**重启 WSL**（`wsl --shutdown` 后重开）生效：

```bash
sudo usermod -aG dialout $USER
```

## 5. 编译 hello_world

用 sysbuild 方式编译（会自动带上 mcuboot 二级引导）：

```bash
cd ~/zephyrproject/zephyr
west build -p always -b xiao_esp32c3 --sysbuild samples/hello_world
```

关键输出：CMake 会先构建 mcuboot，再构建 hello_world，最后生成 ESP32-C3 镜像：

```text
-- Board: xiao_esp32c3, qualifiers: esp32c3
-- Zephyr version: 4.4.99
-- Found toolchain: zephyr 1.0.1 (/home/xx/zephyr-sdk-1.0.1)
...
[239/239] Linking C executable zephyr/zephyr.elf
Memory region         Used Size  Region Size  %age Used
     mcuboot_hdr:          32 B         32 B    100.00%
        metadata:          80 B         96 B     83.33%
           FLASH:      134116 B    4194176 B      3.20%
...
esptool v5.3.1
Successfully created ESP32-C3 image.
```

编译时可能看到一条无害警告：

```text
warning: MCUBOOT_UPDATE_FOOTER_SIZE ... was assigned the value '0x30' but got the value ''
```

这是 sysbuild 默认带了 mcuboot，而 hello_world 没有启用 img_manager 导致 Kconfig 依赖未满足，不影响编译和运行，忽略即可。

## 6. 烧录

```bash
west flash
```

west 会自动选择 `/dev/ttyACM0`，走 USB-Serial/JTAG 直连烧录（921600bps）：

```text
-- west flash: using runner esp32
Detecting chip type... ESP32-C3
Auto-selected /dev/ttyACM0 for esp32c3
Chip type:          ESP32-C3 (QFN32) (revision v0.3)
Features:           Wi-Fi, BT 5 (LE), Single Core, 160MHz, Embedded Flash 4MB (XMC)
Crystal frequency:  40MHz
USB mode:           USB-Serial/JTAG
...
Wrote 32576 bytes at 0x00000000 in 0.4 seconds   # mcuboot
Wrote 134284 bytes at 0x00020000 in 0.7 seconds   # hello_world
Hash of data verified.
Hard resetting via RTS pin...
```

## 7. 串口监视

```bash
west espressif monitor
```

启动后能看到完整启动日志和 Hello World 输出：

```text
--- idf_monitor on /dev/ttyACM0 115200 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
ESP-ROM:esp32c3-api1-20210207
...
I (soc_init): MCUboot 2nd stage bootloader
I (boot): Loading image 0 - slot 0 from flash, area id: 2
*** Booting Zephyr OS build v4.4.0-13771-gb9df9f46ae46 ***
Hello World! xiao_esp32c3/esp32c3
```

看到 `Hello World! xiao_esp32c3/esp32c3` 就说明环境全部打通了。

## 常见问题

1. **WSL 里 lsusb 看不到 ESP32-C3**
   确认在管理员 PowerShell 里执行过 `usbipd bind --busid=<BUSID>`，且 `usbipd attach --wsl --busid=<BUSID>` 成功。**Windows 重启或 WSL 重启后，attach 会失效，需要重新 attach**（bind 一次即可）。

2. **/dev/ttyACM0 权限不足**
   ```text
   could not open port /dev/ttyACM0: Permission denied
   ```
   加入 dialout 组并重启 WSL：`sudo usermod -aG dialout $USER`。

3. **烧录报错 "No such file or directory"（找不到串口）**
   先 `ls /dev/ttyACM*` 确认设备在；如果插了多个开发板，可以指定设备：
   ```bash
   west flash --esp-device /dev/ttyACM1
   ```

4. **west update 慢或失败**
   网络问题，挂代理或换镜像源；国内可考虑配置 `west config` 使用镜像仓库。

5. **exit code 2 之类的中途编译失败**
   大多是依赖没装全，回到第 1 步对照 `apt install` 清单补装，或 `pip install west` 后重跑 `west packages pip --install`。

## 参考

- [Zephyr Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)
- [usbipd-win：WSL2 USB 设备共享](https://github.com/dorssel/usbipd-win)
- [wsl-dashboard：USB 管理 GUI（可选）](https://github.com/owu/wsl-dashboard)
