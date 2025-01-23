---
title: "AI 换脸，Deep-Live-Cam 安装笔记"
date: 2025-01-23T00:49:37+08:00
lastmod: 2025-01-23T00:49:37+08:00
author: ["hacper"]
tags:
    - AIGC
    - Deep-Live-Cam
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

Python (3.10 recommended)
ffmpeg
Visual Studio 2022 Runtimes (Windows)

Clone the Repository
https://github.com/hacksider/Deep-Live-Cam.git

GFPGANv1.4
inswapper_128_fp16.onnx
Place these files in the "models" folder.

We highly recommend using a venv to avoid issues
pip install -r requirements.txt

Install CUDA Toolkit 11.8 or CUDA Toolkit 12.1.1

pip uninstall onnxruntime onnxruntime-gpu
pip install onnxruntime-gpu==1.16.3

python run.py --execution-provider cuda


1. Image/Video Mode

Execute python run.py.
Choose a source face image and a target image/video.
Click "Start".
The output will be saved in a directory named after the target video.
2. Webcam Mode

Execute python run.py.
Select a source face image.
Click "Live".
Wait for the preview to appear (10-30 seconds).
Use a screen capture tool like OBS to stream.
To change the face, select a new source image.
