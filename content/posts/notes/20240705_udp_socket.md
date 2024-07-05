---
title: "udp socket 收不到数据问题"
date: 2024-07-05T02:00:18+08:00
lastmod: 2024-07-05T02:00:18+08:00
author: ["hacper"]
tags:
   - udp
   - scoket
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

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
// #include <arpa/inet.h>
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
    // char send_data[] = "{\"test\":123}";
    char response[100] = {0};

    /* Use the MAKEWORD(lowbyte, highbyte) macro declared in Windef.h */
    wVersionRequested = MAKEWORD(2, 2);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0)
    {
        /* Tell the user that we could not find a usable */
        /* Winsock DLL.                                  */
        printf("WSAStartup failed with error: %d\n", err);
        return 1;
    }

#endif

    // 创建UDP套接字
    if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    // 设置本地地址
    memset(&local_addr, 0, sizeof(local_addr));
    local_addr.sin_family = AF_INET;
    local_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    local_addr.sin_port = htons(PORT);

    // 绑定套接字
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

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
// #include <arpa/inet.h>
#include <unistd.h>
#include <pthread.h>
// #include <arpa/inet.h> // For inet_ntoa()
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
        /* code */
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
        // printf("Received message: %s\n", buffer);
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
    // char send_data[] = "{\"test\":123}";
    char response[100] = {0};

    /* Use the MAKEWORD(lowbyte, highbyte) macro declared in Windef.h */
    wVersionRequested = MAKEWORD(2, 2);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0)
    {
        /* Tell the user that we could not find a usable */
        /* Winsock DLL.                                  */
        printf("WSAStartup failed with error: %d\n", err);
        return 1;
    }

#endif

    // 创建接收线程
    if (pthread_create(&receive_thread, NULL, receive_broadcast, NULL) != 0) {
        perror("pthread_create");
        exit(EXIT_FAILURE);
    }

    // 创建UDP套接字
    if ((sock = socket(AF_INET, SOCK_DGRAM, 0)) < 0) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    // 设置本地地址
    memset(&local_addr, 0, sizeof(local_addr));
    local_addr.sin_family = AF_INET;
    local_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    local_addr.sin_port = htons(3387);

     // 绑定套接字
    if (bind(sock, (struct sockaddr*)&local_addr, sizeof(local_addr)) < 0) {
        perror("bind");
        close(sock);
        pthread_exit(NULL);
    }


    // 允许套接字发送广播消息
    if (setsockopt(sock, SOL_SOCKET, SO_BROADCAST, &broadcast_enable, sizeof(broadcast_enable)) < 0) {
        perror("setsockopt(SO_BROADCAST)");
        close(sock);
        exit(EXIT_FAILURE);
    }

    // 设置广播地址
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
    // 发送组播消息
    while (1) {
        // if (sendto(sock, MESSAGE, strlen(MESSAGE), 0, (struct sockaddr*)&broadcast_addr, sizeof(broadcast_addr)) < 0) {
        if (send(sock, MESSAGE, strlen(MESSAGE), 0) < 0) {
            perror("sendto");
            close(sock);
            exit(EXIT_FAILURE);
        }
        printf("Sent message: %s\n", MESSAGE);
        sleep(2); // 每2秒发送一次
    }

    close(sock);
    return 0;
}


```


coap_client.exe coap://192.168.10.255:5683/qlink/searchRouter 这个有回复socket收不到。
coap_client.exe coap://192.168.10.1:5683/qlink/searchRouter 这个可以收到

![60d7eaff8e5efa6a6970def69f7f49a](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/60d7eaff8e5efa6a6970def69f7f49a.66rpo8sdnvc0.webp)

抓包有收到数据包，但socket一直select超时

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
ql_coap_send_recv timeout

```

对于广播地址，udp 不能使用 connect send read 接口，不然收不到数据。

