---
title: "flowerss bot 安装与迁移"
date: 2023-12-11T00:08:42+08:00
lastmod: 2024-05-07T00:08:42+08:00
author: ["hacper"]
tags:
    - flowerss
    - docker
    - blog
    - rss
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "记录 flowerss bot 的安装与迁移方法" # 文章简单描述，会展示在主页
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

flowerss bot 是一个开源的 Telegram RSS Bot，支持自建和docker部署，目前主要用来订阅自己的博文以及推送到 Telegram 频道。

## 安装


```bash
# 下载配置模板
mkdir ~/flowerss &&\
wget -O ~/flowerss/config.yml \
    https://raw.githubusercontent.com/indes/flowerss-bot/master/config.yml.sample

# 修改配置文件
vim ~/flowerss/config.yml

# 运行
docker run -d --restart=always -v ~/flowerss:/root/.flowerss indes/flowerss-bot
```

注意：
- docker 容器挂载的路径 root/.flowerss 不要修改，不然无法启动。
- 数据库的文件位置放在 /root/.flowerss 路径下，不要修改，方便备份。
```yaml
sqlite:
  path: /root/.flowerss/data.db
```
- bot_token 必须配置。 可以配置 allowed_users 的用户id, 限制只能自己使用。

## 备份 & 迁移

只要备份 ~/flowerss 这个目录，迁移到新的主机也只需要复制 ~/flowerss 下的文件，执行 `docker run -d --restart=always -v ~/flowerss:/root/.flowerss indes/flowerss-bot`安装即可。

## 链接

- [https://github.com/indes/flowerss-bot](https://github.com/indes/flowerss-bot)