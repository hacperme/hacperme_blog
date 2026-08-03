---
title: "绿联 NAS 配置 frp 内网穿透"
date: 2025-08-11T00:28:37+08:00
lastmod: 2025-08-20T00:28:37+08:00
author: ["hacper"]
tags:
    - NAS
    - docker
    - frp
    - DXP2800
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "记录使用 docker+frp 为绿联 NAS 搭建内网穿透服务的一些经验。" # 文章简单描述，会展示在主页
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

绿联 NAS 当前自带内网穿透功能，对于远程访问设备服务很方便，但是自带的内网穿透功能也有缺点，需要登录用户账号之后才能使用，而且映射的域名是动态变化的。对于一些需要固定地址且稳定访问的服务，还是得考虑自建服务穿透。这里记录使用 docker+frp 搭建内网穿透服务的一些经验。

## 服务器

需要一台具备公网ip的服务器，使用 docker 搭建frp 服务端。
新建一个 frps 目录，新建 frp 配置文件 frps.toml：

```toml
bindPort = 7000

transport.maxPoolCount = 15

transport.tls.force = false

webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "12345678"

detailedErrorsToClient = true

auth.method = "token"
auth.token = "12345678"

```

注意需要设置 `webServer.addr = "0.0.0.0"`, 不然打不开web管理面板界面。auth.token 和 webServer.password 按照实际需要修改成自己的。

然后再创建 docker-compose.yaml 文件：

```yaml
services:
  frps:
    image: snowdreamtech/frps
    container_name: frps
    network_mode: bridge
    volumes:
      - ./frps.toml:/frp/frps.toml
    ports:
      # 监听端口
      - "7000:7000"
      # 面板端口
      - "7500:7500"
      # nas
      - "9999:9999"
    command: ["-c", "/frp/frps.toml"]
    restart: always
```

服务器上的 network_mode 设置成 bridge，需要外网访问的端口需要映射到host。包括后面在客户端新增的服务端口，都需要映射到 host， 比如这里的 nas 登录服务 `- "9999:9999"`。

然后执行指令 `docker-compose up -d` 启动 frp 服务器。

## 客户端 NAS

在 NAS 上搭建 frp 客户端需要在共享目录创建 docker/frpc 文件目录，创建配置文件 frpc.toml ：

```toml
serverAddr = "117.125.189.120"
serverPort = 7000


auth.method = "token"
auth.token = "12345678"

[[proxies]]
name = "nas"
type = "tcp"
localIP = "127.0.0.1"
localPort = 9999
remotePort = 9999
```

其中 `serverAddr = "117.125.189.120"` 改成自己的公网服务器地址，`auth.token = "12345678"`改成服务器上配置的 token。
增加需要内网穿透的服务， localIP 设置为 `127.0.0.1`，端口 localPort 和 remotePort 设置为实际使用的端口。

在绿联 NAS docker 应用界面新建项目，保存路径选择 docker/frpc 目录，填写 docker-compose.yaml 内容：


```yaml
services:
  frpc:
    image: snowdreamtech/frpc
    container_name: frpc
    volumes:
      - ./frpc.toml:/frp/frpc.toml
    network_mode: host
    command: ["-c", "/frp/frpc.toml"]
    restart: always
```
![](https://github.com/hacperme/picx-images-hosting/raw/master/20250820/image.102hvxifz5.webp)

然后点立即部署创建容器。

注意 network_mode 需要设置为 host，创建容器、启动客户端之后便可以通过公网服务器ip:端口访问NAS的服务了,如访问 117.125.189.120:9999 登录NAS。

如果有需要，之后可以添加更多的服务穿透，新增、修改、和删除服务只需要修改服务器和客户端的的配置文件，再重新启动 docker 容器。