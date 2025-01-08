---
title: "Github 仓库 ssh 连接失败处理"
date: 2024-12-31T00:49:37+08:00
lastmod: 2024-12-31T00:49:37+08:00
author: ["hacper"]
tags:
    - git
    - ssh
    - github
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

突然 ssh 连不上 github，无法拉取仓库代码，配置 https.proxy http.proxy 代理无效（可能我用的是 ssh 的原因）。然后看到 [git 设置和取消代理](https://gist.github.com/laispace/666dd7b27e9116faece6) 可以尝试将 ssh 端口修改为 443，真救了老命了。


```bash
# 创建 ~/.ssh/config 配置文件和写入如下配置
# 将 ssh 端口修改为 443
Host github.com
    Hostname ssh.github.com
    Port 443
    User git

```
