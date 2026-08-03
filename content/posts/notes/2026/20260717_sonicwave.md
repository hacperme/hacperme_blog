---
title: "Sonic Wave：一个纯浏览器端的音频转换与压缩工具"
date: 2026-07-17T10:22:58+08:00
lastmod: 2026-08-03T10:22:58+08:00
author: ["hacper"]
tags:
    - 音频
    - FFmpeg
    - Rust
    - Docker
    - WebAssembly
    - SonicWave
categories:
    - 笔记
description: "介绍 Sonic Wave 的设计思路、核心实现、部署方式和使用方法。"
summary: "Sonic Wave 是我开发的一个纯浏览器端音频转换与压缩工具，支持多格式转换、批量处理与 Docker 部署，本文记录它的设计思路与使用方法。"
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
---

最近我做了一个小工具项目：[Sonic Wave](https://github.com/hacperme/SonicWave)。它的目标很直接：**把常见音频转换、压缩、批量处理这件事，尽可能简单地放到浏览器里完成。**

在线体验地址：<https://sonic-wave.hacperme.com/>

如果你平时经常遇到下面这些需求，那么这个工具可能正好适合你：

- 把 WAV 转成 MP3 / M4A / OGG / FLAC
- 调整音频比特率、采样率、声道
- 批量处理多个音频文件
- 不想安装桌面软件，只想打开网页马上用
- 不希望把音频上传到第三方服务器

这篇文章记录一下 Sonic Wave 的设计思路、核心实现和使用方法，也顺便给它打个广告（笑）。如果你正好有音频转码、音频压缩、批量格式转换的需求，欢迎试用和 star：

- GitHub：<https://github.com/hacperme/SonicWave>
- 在线网站：<https://sonic-wave.hacperme.com/>

## 项目想解决什么问题

很多在线音频工具虽然方便，但通常有几个问题：

1. 需要把文件上传到服务器
2. 对隐私敏感内容不友好
3. 批量处理体验一般
4. 页面广告多、速度不稳定
5. 部署和迁移不方便

Sonic Wave 的思路是反过来：**服务器只负责把页面和静态资源发给浏览器，真正的音频处理全部在客户端完成。**

这样做的好处很明显：

- **隐私更好**：音频文件不需要上传到后端
- **部署更轻**：后端不承担转码计算压力
- **可移植性更强**：一个静态前端 + 一个轻量服务端就能跑起来
- **更适合自部署**：个人网站、NAS、小服务器都可以挂起来

## Sonic Wave 能做什么

目前 Sonic Wave 主要支持这些能力：

- 支持 **MP3、WAV、AAC/M4A、OGG Vorbis、FLAC** 格式转换
- 支持 **批量选择多个音频文件**
- 支持设置：
  - 输出格式
  - 声道（单声道 / 立体声）
  - 采样率
  - 比特率
- 支持批量处理完成后 **打包 ZIP 下载**
- 支持音频基础信息解析，例如：
  - 时长
  - 采样率
  - 声道
  - 原始比特率

从页面交互上看，整个流程也比较直接：

1. 拖拽或选择音频文件
2. 选择输出格式和参数
3. 点击开始处理
4. 逐个下载，或者打包下载全部结果

## 设计思路

Sonic Wave 的架构其实很克制，可以概括成一句话：

> **前端负责所有音频处理，后端只做静态资源分发和浏览器运行环境保障。**

### 前端部分：FFmpeg.wasm 做核心处理

项目前端是一个单页面应用，主要逻辑写在 `index.html` 中，核心依赖是本地化的 FFmpeg.wasm 资源：

- `assets/ffmpeg.js`
- `assets/ffmpeg-core.js`
- `assets/ffmpeg-core.wasm`
- `assets/util.js`
- `assets/jszip.min.js`

页面初始化时会加载本地 FFmpeg 核心，而不是依赖外部 CDN。这样做有两个好处：

- 部署时更可控，不依赖第三方资源可用性
- 更适合内网、NAS 或自建服务场景

在转换时，前端会把用户选择的文件写入 FFmpeg 的虚拟文件系统，然后按选项拼出命令，例如：

- `-ac` 控制声道
- `-ar` 控制采样率
- `-b:a` 控制音频比特率
- `-acodec` 选择对应输出编码器

处理完成后，再从 WASM 文件系统中读取输出结果，转成浏览器可下载的 `Blob`。

### 为什么强调“纯浏览器端”

因为这不是简单地在网页上加一个上传按钮，然后服务器上跑 ffmpeg。Sonic Wave 的关键点在于：

- 音频文件在浏览器本地处理
- 服务端不接收待转换的音频内容
- 转换、压缩、打包下载都尽量在客户端完成

对于隐私敏感的录音、会议音频、学习资料、个人素材，这一点很重要。

## 一些实现细节

这个项目里有几处我觉得比较实用的设计。

### 1. 元数据解析先行

在真正转换前，Sonic Wave 会先尝试解析每个文件的基础信息。做法是调用 FFmpeg 的 `-i` 行为读取输入文件信息，再从日志里提取：

- Duration
- Sample Rate
- Channels
- Bitrate

这样用户在开始处理前，就能先看到文件的大概情况，避免“选完就转，但其实参数完全不清楚”的盲操作。

### 2. 批量处理时考虑了内存回收

浏览器里跑 FFmpeg.wasm，最现实的问题之一就是内存。

Sonic Wave 目前在批量处理时采用的是**串行处理**，并且每处理一批文件会主动重新初始化 FFmpeg 实例来释放内存。项目里还做了这些处理：

- 每个文件处理完成后立即删除临时输入 / 输出文件
- 读取结果后立刻复制到独立内存，避免长期占用 WASM 内存
- 批量处理时带简单重试机制
- 对常见错误给出更友好的中文提示

这类设计没有追求“同时并发处理一堆文件”，而是优先保证稳定性和可用性。对于浏览器端大文件转码来说，我觉得这是更实际的路线。

### 3. 文件大小限制更明确

为了避免浏览器直接被大文件拖垮，页面上限制了：

- 单文件不超过 **100MB**
- 总文件大小不超过 **500MB**

这不一定适合所有场景，但对于在线工具来说，这是一个比较务实的折中。

### 4. 支持批量结果打包下载

如果一次转换了多个文件，Sonic Wave 会提供 **ZIP 打包下载**。这点对实际使用体验帮助很大，不然一个个点下载会很烦。

### 5. 对 WebAssembly 运行环境做了专门处理

为了让 FFmpeg.wasm 正常工作，服务端在响应里增加了：

- `Cross-Origin-Opener-Policy: same-origin`
- `Cross-Origin-Embedder-Policy: require-corp`

在线站点当前返回头里也能看到这两个配置。这类 Header 虽然不显眼，但对某些 WebAssembly / SharedArrayBuffer 相关能力是很关键的。

## 后端为什么用 Rust

Sonic Wave 的后端并不复杂，但我还是选了 Rust + Axum 来实现一个轻量静态文件服务。

它主要负责几件事：

- 提供静态资源访问
- 注入 COOP / COEP 响应头
- 区分 HTML 和静态资源的缓存策略
- 读取配置文件和环境变量
- 提供更适合生产部署的运行方式

项目里的配置优先级是：

> **环境变量 > `config.toml` > 默认值**

默认配置大致如下：

- 端口：`8089`
- 静态目录：`.`
- HTML：`no-cache, must-revalidate`
- 静态资源：`public, max-age=31536000, immutable`

这个设计很适合部署静态 Web 工具：

- HTML 不强缓存，更新能更快生效
- JS / WASM / CSS 这类资源可以长期缓存，减轻带宽压力

## Docker 部署也做得比较轻

这个项目除了可以直接运行二进制，还支持 Docker 部署。

Dockerfile 用的是**多阶段构建**：

1. 在 Rust 构建镜像里编译服务端
2. 在精简 Alpine 镜像里只放最终产物
3. 用非 root 用户运行
4. 自带健康检查

如果你只是想快速部署，最方便的方式就是直接用 `docker-compose.yml`。

## 使用方法

### 在线使用

最简单的方式就是直接打开：

<https://sonic-wave.hacperme.com/>

使用步骤：

1. 打开网页
2. 拖拽音频文件，或点击上传
3. 选择输出格式
4. 按需调整比特率、采样率、声道
5. 点击开始处理
6. 下载转换结果

适合这些场景：

- 把录音转成更通用的 MP3
- 把音频体积压小，方便分享
- 批量整理不同格式的音频文件
- 快速导出适合手机、播放器或网页使用的格式

### 本地 / 服务器部署

#### 方式一：Docker Compose

docker-compose.yml 文件：

```yaml
services:
  sonic-wave:
    image: hacper/sonic-wave:latest
    ports:
      - "18099:8089"
    environment:
      - PORT=8089
      - STATIC_DIR=/app
    volumes:
      # 可选：挂载自定义配置文件
      - ./config.toml:/app/config.toml:ro
    restart: unless-stopped
    container_name: sonic-wave
```

在根目录执行：

```bash
docker-compose up -d
```

默认会监听 `8089` 端口，然后访问：

```text
http://localhost:8089
```

#### 方式二：Docker 手动运行

```bash
docker build -t sonic-wave .
docker run -d -p 8089:8089 --name sonic-wave sonic-wave
```

#### 方式三：直接运行可执行文件

先编译：

```bash
cargo build --release
```

然后运行：

```bash
./target/release/sonic-wave
```

### 配置文件示例

可以通过 `config.toml` 自定义端口、静态目录和缓存策略：

```toml
port = 8089
static_dir = "."
cache_control = "public, max-age=31536000, immutable"
html_cache_control = "no-cache, must-revalidate"
```

也可以直接通过环境变量覆盖：

```bash
PORT=9000 STATIC_DIR=. ./target/release/sonic-wave
```

## 适合谁用

我觉得 Sonic Wave 特别适合下面几类用户：

- 想要一个**自部署在线音频工具**的人
- 希望文件**不经过后端上传**的人
- 需要**批量转换音频**的人
- 想在 NAS、轻量云服务器、个人站点里挂一个实用工具的人
- 想研究 **FFmpeg.wasm + Rust 静态服务** 组合的人

## 目前的一些边界

当然，这个项目也不是没有边界。

比如浏览器端工具天然受这些因素影响：

- 浏览器内存限制
- 设备性能差异
- 大文件长音频处理时间较长
- 某些编码格式在不同浏览器环境下体验不同

所以 Sonic Wave 更适合：

- 日常轻量到中等规模的音频处理
- 格式转换
- 批量整理
- 快速压缩和导出

如果是超大体积、超长时长、专业级流水线处理，那还是本地原生 FFmpeg 更合适。

## 为什么我还挺推荐你试试

因为它确实是一个比较“顺手”的工具：

- 打开就能用
- 不需要安装复杂软件
- 不需要把文件传给别人
- 对自部署很友好
- 技术栈也比较清晰，适合二次修改

如果你平时正好会遇到：

- 手机录音太大
- 音频格式不兼容
- 想批量转成 MP3 / M4A
- 想把一个实用小工具挂到自己的网站上

那 Sonic Wave 值得收藏一下：

- 在线体验：<https://sonic-wave.hacperme.com/>
- 项目地址：<https://github.com/hacperme/SonicWave>

如果你用了之后觉得顺手，也欢迎给个 star，或者按自己的需求继续改造成更适合自己的版本。

## 小结

Sonic Wave 不是一个追求“大而全”的音频平台，而是一个目标明确的小工具：

> **用尽量简单的架构，提供一个可在线使用、可自部署、重隐私、支持批量处理的音频转换工具。**

我自己挺喜欢这类项目：功能具体、结构清晰、部署不重、拿来就能用。

如果你也喜欢这种风格，欢迎试试 Sonic Wave。
