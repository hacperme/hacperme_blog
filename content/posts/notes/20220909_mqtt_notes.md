---
title: "MQTT 应用笔记"
date: 2022-09-09T22:09:22+08:00
lastmod: 2023-05-14T01:19:57+08:00
author: ["hacper"]
tags:
    - MQTT
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

## cleansession 与 Qos 的配置关系

1. cleanSession的语义

   **cleanSession**标志是MQTT协议中对一个消费者客户端建立TCP连接后是否关心之前状态的定义，与消息发送端的设置无关。具体语义如下： - **cleanSession=true**：消费者客户端再次上线时，将不再关心之前所有的订阅关系以及离线消息。（服务器不保存之前的topic历史关系和不存储离线消息，客户连接需要重新订阅topic且不会收到离线消息） - **cleanSession=false**：消费者客户端再次上线时，还需要处理之前的离线消息，而之前的订阅关系也会持续生效。（服务器会保存之前的topic订阅关系，客户端连接之后不需要重新订阅topic，且会收到离线消息）

2. cleanSession 与 Qos 配置的关系

| QoS级别 | cleanSession=true                  | cleanSession=false                                           |
| :------ | :--------------------------------- | :----------------------------------------------------------- |
| QoS0    | 无离线消息，在线消息只尝试推一次。 | 无离线消息，在线消息只尝试推一次。                           |
| QoS1    | 无离线消息，在线消息保证可达。     | 有离线消息，所有消息保证可达。                               |
| QoS2    | 无离线消息，在线消息保证只推一次。 | 有离线消息，所有消息保证只推一次。（也需要服务器支持该功能才有效） |



## 发布、订阅消息不同 Qos 配置的作用、交互流程及其特点



1. mqtt 消息发布者和消息订阅者之间的 Qos 配置关系


MQTT发布消息QoS保证不是端到端的，是客户端与服务器之间的。订阅者收到MQTT消息的QoS级别，最终取决于发布消息的QoS和主题订阅的QoS。


| 发布消息的 QoS | 主题订阅的 QoS | 接收消息的 QoS |
| -------------- | -------------- | -------------- |
| 0              | 0              | 0              |
| 0              | 1              | 0              |
| 0              | 2              | 0              |
| 1              | 0              | 0              |
| 1              | 1              | 1              |
| 1              | 2              | 1              |
| 2              | 0              | 0              |
| 2              | 1              | 1              |
| 2              | 2              | 2              |




2. 不同 Qos 消息在消息发布者、服务器、消息订阅者之间的交互流程

   

- Qos0 的交互流程

![mqtt-qos0-2022-08-11-1819](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/mqtt-qos0-2022-08-11-1819.3pyah3ua5t00.png)

特点：

1. 没有重发机制，消息订阅者不一定能收到消息。



- Qos1的交互流程

![mqtt-qos1-2022-08-11-1819](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/mqtt-qos1-2022-08-11-1819.1qw727hrf20w.png)

特点：

1. 有重传机制，消息发送者PUBLISH消息之后，在特定时间之内没有收到 PUBACK 确认消息，就会触发重传。
2. 重传对象为 PUBLISH 报文，重传动作在发送端进行。
3. 一般重传有次数限制，比如腾讯云 iot 平台，3s 内没有收到ACK触发重传，重传次数最大为3次。
4. 由于有重传，在应用层有可能会收到重复消息，需要根据应用场景判断要不要增加消息去重处理。



- Qos2的交互流程

![mqtt-qos2-2022-08-11-1819](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/mqtt-qos2-2022-08-11-1819.72y1jwbvcd80.png)

特点：

1. Qos2 不一定服务器都支持。
2. Qos2 发送消息，发送端和接收端需要个发两次报文才完成整个流程，共4次报文传输，流程更复杂，开销相对较大。
3. 重传动作发生在消息发送端，重传对象有 PUBLISH 报文和 PUBREL 报文。
4. 虽然有重传，但是应用层不会收到重复消息。



## 搭建 mqtt 测试服务器



- 搭建 emqx 

推荐使用 docker 的方式，部署简单，一条命令搞定。

```bash
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:latest
```
