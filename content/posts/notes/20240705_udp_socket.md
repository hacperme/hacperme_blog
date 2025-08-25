---
title: "udp socket 收不到数据问题"
date: 2024-07-05T02:00:18+08:00
lastmod: 2024-07-14T02:00:18+08:00
author: ["hacper"]
tags:
   - udp
   - scoket
   - coap
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "排查 udp socket 收不到数据问题，又长脑子啦。" # 文章简单描述，会展示在主页
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

最近在调试一个 coap 发送请求接收不到响应数据的问题，客户端往局域网内的广播地址发送请求收不到响应，但是往局域网的实际设备的地址发送请求却能收到响应。


coap://192.168.10.255:5683/qlink/searchRouter 广播地址收不到响应，一直 socket select 超时。
```bash
libcoap on  dev [!+?] via △ v3.19.1 via 🌙 
❯ .\build\mingw\x86_64\release\coap_client.exe coap://192.168.10.255:5683/qlink/searchRouter 
uri:coap://192.168.10.255:5683/qlink/searchRouter
send_data:{"searchKey":"ANDLINK-DEVICE","andlinkVersion":"V3"}
is_mcast 0
create socket2:592
bind2!!!!!!!!!!!
Jul 04 16:29:43.593 DEBG ***192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : session 00007ff718508994: created outgoing session
Jul 04 16:29:43.599 DEBG ***192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : session connected
v:1 t:CON c:POST i:cbb3 {01} [ Uri-Path:qlink, Uri-Path:searchRouter, Content-Format:application/json ] :: '{"searchKey":"ANDLINK-DEVICE","andlinkVersion":"V3"}'
Jul 04 16:29:43.609 DEBG *  192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : netif: sent   86 bytes
v:1 t:CON c:POST i:cbb3 {01} [ Uri-Path:qlink, Uri-Path:searchRouter, Content-Format:application/json, Request-Tag:0xd13222ab ] :: '{"searchKey":"ANDLINK-DEVICE","andlinkVersion":"V3"}'
Jul 04 16:29:43.621 DEBG ** 192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : mid=0xcbb3: added to retransmit queue (2969ms)
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
select:0
nfds:593
nfds:593
select:0
Jul 04 16:29:46.597 DEBG ***EVENT: COAP_EVENT_MSG_RETRANSMITTED
Jul 04 16:29:46.600 DEBG ** 192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : mid=0xcbb3: retransmission #1 (next 5938ms)
Jul 04 16:29:46.604 DEBG *  192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : netif: sent   86 bytes
v:1 t:CON c:POST i:cbb3 {01} [ Uri-Path:qlink, Uri-Path:searchRouter, Content-Format:application/json, Request-Tag:0xd13222ab ] :: '{"searchKey":"ANDLINK-DEVICE","andlinkVersion":"V3"}'
select:0
Jul 04 16:29:46.722 ERR  timeout
Jul 04 16:29:46.724 DEBG ***192.168.10.3:12345 <-> 192.168.10.255:5683 UDP : session 00007ff718508994: closed

```

对比正常的，coap://192.168.10.1:5683/qlink/searchRouter 这个地址可以收到响应，程序相同只是请求地址不同。

```bash
libcoap on  dev [!+?] via △ v3.19.1 via 🌙 took 2s
❯ .\build\mingw\x86_64\release\coap_client.exe coap://192.168.10.1:5683/qlink/searchRouter   
uri:coap://192.168.10.1:5683/qlink/searchRouter
send_data:{"searchKey":"ANDLINK-DEVICE","andlinkVersion":"V3"}
is_mcast 0
create socket2:604
bind2!!!!!!!!!!!
Jul 04 16:29:32.373 DEBG ***192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : session 00007ff718508994: created outgoing session
Jul 04 16:29:32.381 DEBG ***192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : session connected
v:1 t:CON c:POST i:1757 {01} [ Uri-Path:qlink, Uri-Path:searchRouter, Content-Format:application/json ] :: '{"searchKey":"ANDLINK-DEVICE","andlinkVersion":"V3"}'
Jul 04 16:29:32.392 DEBG *  192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : netif: sent   86 bytes
ICE","andlinkVersion":"V3"}'
Jul 04 16:29:32.406 DEBG ** 192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : mid=0x1757: added to retransmit queue (2219ms)
nfds:605
select:1
Jul 04 16:29:32.413 DEBG *  192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : netif: recv   60 bytes
v:1 t:ACK c:2.05 i:1757 {01} [ Content-Format:application/json ] :: '{"searchAck":"ANDLINK-ROUTER","andlinkVersion":"V4"}'
Jul 04 16:29:32.422 DEBG ** 192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : mid=0x1757: removed (1)
have_response
v:1 t:ACK c:2.05 i:1757 {01} [ Content-Format:application/json ] :: '{"searchAck":"ANDLINK-ROUTER","andlinkVersion":"V4"}'
recv[52]:{"searchAck":"ANDLINK-ROUTER","andlinkVersion":"V4"}
Jul 04 16:29:32.437 DEBG ***192.168.10.3:12345 <-> 192.168.10.1:5683 UDP : session 00007ff718508994: closed
recv rsp:{"searchAck":"ANDLINK-ROUTER","andlinkVersion":"V4"}

```
![](https://github.com/hacperme/picx_hosting/raw/master/20210507/60d7eaff8e5efa6a6970def69f7f49a.66rpo8sdnvc0.webp)

抓包分析看到服务器有回复，但 socket 就是收不到数据，百思不得其解。

暂时没有什么分析思路，于是打算写个简单的 udp demo 在两台电脑上运行验证看看，一台电脑往广播地址发生数据，另一台电脑一直监听 socket 收数据, 收到数据再往源地址发一个回复的测试数据，然后看向广播地址发送数据的电脑能否收到数据。验证下来又是正常的，两边都能收发数据。

后面再看了一遍 coap 代码的执行流程，创建 udp socket 之后调用了 connect，收发数据使用 send read 接口，而不是 sendto recvfrom，怀疑差别在这里，于是修改测试程序，再验证一下。

往广播地址发送数据的测试程序：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
#ifdef _WIN32
#include <windows.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#endif

#define BROADCAST_IP "192.168.10.255"
#define PORT 12345
#define MESSAGE "Hello, Broadcast!"
int sock = -1;
int sock_ok = 0;

void *receive_broadcast(void *arg) {
  
    struct sockaddr_in local_addr;
    char buffer[256];
    int addr_len;
    int message_len;

    while (sock_ok == 0)
    {
    }

    while (1) {
        addr_len = sizeof(local_addr);
        // message_len = recvfrom(sock, buffer, sizeof(buffer) - 1, 0, (struct sockaddr*)&local_addr, &addr_len);
        message_len = recv(sock, buffer, sizeof(buffer) - 1, 0);
        if (message_len < 0) {
            perror("recvfrom");
            close(sock);
            pthread_exit(NULL);
        }

        buffer[message_len] = '\0';
        printf("Received message from %s: %s\n", inet_ntoa(local_addr.sin_addr), buffer);
    }

    close(sock);
    pthread_exit(NULL);
}

int main() {
    struct sockaddr_in broadcast_addr;
    int broadcast_enable = 1;
    pthread_t receive_thread;
    struct sockaddr_in local_addr;

    #ifdef _WIN32
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;
    char response[100] = {0};
    wVersionRequested = MAKEWORD(2, 2);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0)
    {
        printf("WSAStartup failed with error: %d\n", err);
        return 1;
    }

#endif

    if (pthread_create(&receive_thread, NULL, receive_broadcast, NULL) != 0) {
        perror("pthread_create");
        exit(EXIT_FAILURE);
    }
    if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket");
        exit(EXIT_FAILURE);
    }
    memset(&local_addr, 0, sizeof(local_addr));
    local_addr.sin_family = AF_INET;
    local_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    local_addr.sin_port = htons(3387);
    if (bind(sock, (struct sockaddr*)&local_addr, sizeof(local_addr)) < 0) {
        perror("bind");
        close(sock);
        pthread_exit(NULL);
    }
    if (setsockopt(sock, SOL_SOCKET, SO_BROADCAST, &broadcast_enable, sizeof(broadcast_enable)) < 0) {
        perror("setsockopt(SO_BROADCAST)");
        close(sock);
        exit(EXIT_FAILURE);
    }
    memset(&broadcast_addr, 0, sizeof(broadcast_addr));
    broadcast_addr.sin_family = AF_INET;
    broadcast_addr.sin_addr.s_addr = inet_addr(BROADCAST_IP);
    broadcast_addr.sin_port = htons(PORT);
    
	if(connect(sock, (struct sockaddr*)&broadcast_addr, sizeof(broadcast_addr)) != 0)
    {
        perror("connect");
        close(sock);
        exit(EXIT_FAILURE);
    }
    sock_ok = 1;
    while (1) {
        // if (sendto(sock, MESSAGE, strlen(MESSAGE), 0, (struct sockaddr*)&broadcast_addr, sizeof(broadcast_addr)) < 0) {
        if (send(sock, MESSAGE, strlen(MESSAGE), 0) < 0) {
            perror("sendto");
            close(sock);
            exit(EXIT_FAILURE);
        }
        printf("Sent message: %s\n", MESSAGE);
        sleep(2);
    }

    close(sock);
    return 0;
}
```

接收数据并回复：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
#ifdef _WIN32
#include <windows.h>
#include <winsock2.h>
#include <ws2tcpip.h>
#endif

#define BROADCAST_IP "192.168.10.255"
#define PORT 12345
#define MESSAGE "hi, Broadcast!"


int main() {

    int sock;
    struct sockaddr_in local_addr;
    char buffer[256];
    int addr_len;
    int message_len;

    #ifdef _WIN32
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;
    char response[100] = {0};
    wVersionRequested = MAKEWORD(2, 2);
    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0)
    {                           */
        printf("WSAStartup failed with error: %d\n", err);
        return 1;
    }

#endif
    if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    memset(&local_addr, 0, sizeof(local_addr));
    local_addr.sin_family = AF_INET;
    local_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    local_addr.sin_port = htons(PORT);

    if (bind(sock, (struct sockaddr*)&local_addr, sizeof(local_addr)) < 0) {
        perror("bind");
        close(sock);
        exit(EXIT_FAILURE);
    }

    while (1) {
        addr_len = sizeof(local_addr);
        message_len = recvfrom(sock, buffer, sizeof(buffer) - 1, 0, (struct sockaddr*)&local_addr, &addr_len);
        if (message_len < 0) {
            perror("recvfrom");
            close(sock);
            exit(EXIT_FAILURE);
        }

        buffer[message_len] = '\0';
        printf("Received message from %s: %s\n", inet_ntoa(local_addr.sin_addr), buffer);
        if (sendto(sock, MESSAGE, strlen(MESSAGE), 0, (struct sockaddr*)&local_addr, sizeof(local_addr)) < 0) {
            perror("sendto");
            close(sock);
            exit(EXIT_FAILURE);
        }
    }

    close(sock);
    return 0;
}
```

结果还真复现了问题，网络抓包有回复，但 socket 收不到数据。

udp socket 收发数据可以使用两套接口: sendto recvfrom 和 connect send recv。udp 使用 connect send recv 可以简化一些编程的处理，适用于与对方是固定地址的通信，在目的地址是广播、组播地址，或者是作为服务需要监听数据的时候就不适合使用 connect 的方式。
