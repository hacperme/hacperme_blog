---
title: "http 学习笔记"
date: 2023-03-06T00:47:35+08:00
lastmod: 2023-03-06T00:47:35+08:00
author: ["hacper"]
tags:
    - tag1
categories:
    - 生活
    - 阅读
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
# cover:
#     image: ""
#     caption: ""
#     alt: ""
#     relative: false

---



## http 协议相关学习资料



- IETF RFC 标准文档

  http 1.1 标准文档

  [RFC2616 ](https://www.rfc-editor.org/rfc/rfc2616.html)和 [RFC7230](https://www.rfc-editor.org/rfc/rfc7230.html)~[RFC7235](https://www.rfc-editor.org/rfc/rfc7235.html)

  http 2 标准文档

  [RFC7540](https://www.rfc-editor.org/rfc/rfc7540.html)

  http 3 标准文档

  [RFC9000](https://www.rfc-editor.org/rfc/rfc9000.html)

  URI 标准文档

  [RFC3986](https://www.rfc-editor.org/rfc/rfc3986.html)

- MDN 文档

  [http 教程 ](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

- 书

  《图解HTTP》



## http 协议发展历程及特点

![123](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvatOuue8A7K8MQAlFoCOtDvWnvOermFicEiaDXia4lnGho0CVXCVAqjOakxAluLcrPVdYM44kqUQeKX2w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

### http /0.9



最初的http 协议没有版本号，为了与后来演进的http 协议作区分，早期的http 协议叫做 http /0.9

自 Tim Berners-Lee 博士和他的团队在1989-1991年

背景

​	蒂姆·伯纳斯·李（Tim Berners-Lee）爵士：

- 万维网（World Wide Web）的发明者
- 《Information Management: A Proposal》https://www.w3.org/History/1989/proposal.html
- https://worldwideweb.cern.ch/history/
- 1990年10月，伯纳斯-李编写了网页的三大基本技术：
  	HTML（超文本标记语言），也就是页面的的标记（格式化）语言。
  	URI（统一资源标识符），一种用来标识各网络资源的唯一“地址”，通常也称作URL。
  	HTTP（超文本传输协议），用于实现链接资源在网络上的提取。

来自世界各地的科学家前来使用CERN的加速器,这些科学家很难共享信息：不同的电脑上有不同的信息，要获得不同信息，科学家就必须登录不同的电脑。而且，有时他们还得在不同的电脑上学习不同的程序。伯纳斯-李认为他可以找到解决这个问题的方法。于是，他提出了一个基于互联网的超链接系统，允许不同计算机之间进行信息共享，而这一想法也永远地改变了人们交流和使用网络的方式。这个系统就是后来人们熟知的万维网（World Wide Web，缩写为www）。

1989年3月，伯纳斯-李在一份名为《信息管理：一个提案》的文件中阐述了他对未来网络的设想。这个提案的目的最初是为了帮助来自世界各地的学者运行复杂的粒子加速器。



特点

- http /0.9 非常简单，功能有限，又称单行协议

- 仅支持GET 请求

- 仅能够传输 html 文本

- 建立与tcp/ip 协议之上

报文例子：

请求只有一行

```http
GET /mypage.html
```

响应只包含文档本身

```html
<HTML>
这是一个非常简单的HTML页面
</HTML>
```



### http /1.0

文档 RFC 1945 定义了 HTTP/1.0，只是一份参考文档，但不是官方标准。

特点：

- 增加 HEAD、POST 等新方法
- 增加响应状态码（status code），标记可能的错误原因
- 引入了协议版本号概念
- 引入了 HTTP Header 的概念，让 HTTP 处理请求和响应更加灵活和更具扩展性
- 传输的数据不再限于文本
- 短连接、无状态

报文示例

```http
GET /mypage.html HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)
```



```http
200 OK
Date: Tue, 15 Nov 1994 08:12:31 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/html
<HTML>
一个包含图片的页面
  <IMG SRC="/myimage.gif">
</HTML>
```



### http /1.1

http /1.1 版本是第一个http 标准化协议，HTTP/1.1 在1997年1月以 [RFC 2068](https://datatracker.ietf.org/doc/html/rfc2068) 文件发布。HTTP/1.1协议进行过两次修订，[RFC 2616](https://datatracker.ietf.org/doc/html/rfc2616) 发布于1999年6月，后面拆分为 [RFC 7230](https://datatracker.ietf.org/doc/html/rfc7230)-[RFC 7235](https://datatracker.ietf.org/doc/html/rfc7235) ，发布于2014年6月，作为HTTP/2的预览版本。

特点：

- 增加了PUT、DELETE、OPTIONS 等新的方法；
- 增加了缓存管理和控制；
- 明确了连接保持（keep-alive），允许持续连接；
- 允许响应数据分块（chunked），利于传输大文件；
- 强制要求 Host 头，能够使不同域名配置在同一个IP地址的服务器上，虚拟主机；
- 引入内容协商机制，包括语言，编码，类型等，并允许客户端和服务器之间约定以最合适的内容进行交换。



报文

```http
GET /en-US/docs/Glossary/Simple_header HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/en-US/docs/Glossary/Simple_header
```



```http
200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Wed, 20 Jul 2016 10:55:30 GMT
Etag: "547fa7e369ef56031dd3bff2ace9fc0832eb251a"
Keep-Alive: timeout=5, max=1000
Last-Modified: Tue, 19 Jul 2016 00:59:33 GMT
Server: Apache
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding

(content)
```



### http /2





### http /3

基于TCP实现的HTTP2遗留下3个问题：

- 有序字节流引出的队头阻塞（Head-of-line blocking），使得HTTP2的多路复用能力大打折扣；
- TCP与TLS叠加了握手时延，建链时长还有1倍的下降空间；
- 基于TCP四元组确定一个连接，这种诞生于有线网络的设计，并不适合移动状态下的无线网络，这意味着IP地址的频繁变动会导致TCP连接、TLS会话反复握手，成本高昂。

HTTP3协议解决了这些问题：

- HTTP3基于UDP协议重新定义了连接，在QUIC层实现了无序、并发字节流的传输，解决了队头阻塞问题（包括基于QPACK解决了动态表的队头阻塞）；
- HTTP3重新定义了TLS协议加密QUIC头部的方式，既提高了网络攻击成本，又降低了建立连接的速度（仅需1个RTT就可以同时完成建链与密钥协商）；
- HTTP3 将Packet、QUIC Frame、HTTP3 Frame分离，实现了连接迁移功能，降低了5G环境下高速移动设备的连接维护成本。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7FCroQRCyibz8UupSjqSHzF6Vu3ciavAPtqK1jZicHOUprMW4alYsbW8icA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)



// todo 绘制各个版本的http 协议架构图

## http 协议调试工具



## http /1.1 协议内容



## https 相关内容



## http /2 相关内容



## http /3 相关内容

与HTTP/2相比，HTTP/3中有哪些重大改进？

![图片](https://mmbiz.qpic.cn/mmbiz_png/CZeKj44ymYaEvSmnRxibfgJa9TJpwcQ0Osp1llyia9mlBibTwsicibFibJiazSXONF9Y1DbXLhWJH7sibAXkBiaQJwam9eg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先，QUIC拥有更快的连接设置，原因是它将传输层握手（TCP的SYN+SYN/ACK+ACK bootstrap）和加密握手（TLS设置）合并到一次RTT中（而HTTP/2需要2~3个RTT）。除此之外，HTTP/3可以受益于TLS 1.3的0-RTT特性，意味着可以发送HTTP请求，并能够在第一次握手期间接收到（部分）响应。尤其当客户端和服务器在地理上相距遥远时，快速连接十分有帮助。

![图片](https://mmbiz.qpic.cn/mmbiz_png/CZeKj44ymYaEvSmnRxibfgJa9TJpwcQ0O7zBIlRnUfmGtLkusFcxmjgA9nofS7YLt5XwzQLZYanbklH0PlsOf4w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/CZeKj44ymYaEvSmnRxibfgJa9TJpwcQ0ONZD8b8z2kE9U7uIumadpxhriaAjGXt2osPtJ46RVaanv505ibqROtngg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

其次，QUIC使用了HTTP/2中的多路复用概念，并将它应用到传输协议层。这一点在技术上不太好理解，不过结果就是QUIC（也指HTTP/3）对于丢包和重新排序的恢复能力得到了提升。这解决了TCP长期存在的“队头阻塞”问题：其中HTTP/2连接上的一个丢包就可能导致其后的数据在等待重传的时候遭到阻塞。在某些情况下，HTTP/3可以解决某些数据的阻塞，以便更早地处理、提交或者执行这些数据。

再次，QUIC还有很多其他性能上的优势。比如，新的“连接迁移”特性将使切换网络（如从Wi-Fi切换到4G）更加无缝（虽然目前少有部署支持）。QUIC在如何确认接收的数据包方面也更加智能，从而更容易决定如何/何时重传丢失的数据包并调整拥塞控制逻辑。这一优势对于HTTP/3并没有太大用处，但是它可以允许其他协议（比如WebTransport或者MASQUE代理）在QUIC之上运行。



tcp 协议的限制：

- TCP仅定义了40字节的可选位，且几乎全部填满。结果就是，没有新特性的位置了。
- 许多中间件（如防火墙）假设TCP数据包将以某种确定方式构造。如果数据包与它们的预期相差太大，就会被拒绝或者延迟，这使得TCP协议几乎无法发展。
- 由于TCP在内核里实现，那么任何TCP传输的更新都需要经过新的内核修改。对于一些基础设施相对陈旧的公司来说，需要耗费数年才能采用新的特性。
- TCP是传输层，没有内置加密（即TLS），所以它需要在上层增加。导致的结果就是需要很长时间才能建立安全连接，并且一些通过TCP传输的数据（比如数据包头部）没有被加密，从而产生安全漏洞。

QUIC 的解决思路：

- 建立在udp之上，UDP在互联网上被广泛部署，所以无需从零开始定义传输层
- 相较于TCP，UDP的开销要少很多，这个特点使它更快速、更简单也更高效。
- QUIC继承了TCP的特性，QUIC同时包含了应用层和传输层机制
- QUIC使用UDP作为底层传输协议，同时内置TLS加密，并结合了TCP的可靠性相关特性。QUIC在应用层（即用户空间）获得进一步实现。因此，无需更新内核，你就可以进行大量修改。

几个重要的QUIC特性：

- 0 RTT建立连接

  RTT: round-trip time, 仅包括请求访问来回的时间

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7Bo3iciciaT8nKhTtB3kAgOiakLickO8aibTDIR2YF5HEcq7unvChs3icf0zCg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  HTTP/2的连接建立需要3 RTT, 如果考虑会话复用, 即把第一次握手计算出来的对称密钥缓存起来, 那也需要2 RTT. 更进一步的, 如果TLS升级到1.3, 那么HTTP/2连接需要2RTT, 考虑会话复用需要1RTT. 如果HTTP/2不急于HTTPS, 则可以简化, 但实际上几乎所有浏览器的设计都要求HTTP/2需要基于HTTPS.

  HTTP/3首次连接只需要1RTT, 后面的链接只需要0RTT, 意味着客户端发送给服务端的第一个包就带有请求数据, 其主要连接过程如下:

  - 首次连接, 客户端发送Inchoate Client Hello, 用于请求连接;

  - 服务端生成g, p, a, 根据g, p, a算出A, 然后将g, p, A放到Server Config中在发送Rejection消息给客户端.

  - 客户端接收到g,p,A后, 自己再生成b, 根据g,p,a算出B, 根据A,p,b算出初始密钥K, B和K算好后, 客户端会用K加密HTTP数据, 连同B一起发送给服务端.

  - 服务端接收到B后, 根据a,p,B生成与客户端同样的密钥, 再用这密钥解密收到的HTTP数据. 为了进一步的安全(前向安全性), 服务端会更新自己的随机数a和公钥, 在生成新的密钥S, 然后把公钥通过Server Hello发送给客户端. 连同Server Hello消息, 还有HTTP返回数据.

    ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7bzwRkicjI9w0m3SibEg7Zt8RowQ6RIgkhiceuicGGiaEvMGGmVrEMicnM2fg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    这里使用DH密钥交换算法, DH算法的核心就是服务端生成a,g,p3个随机数, a自己持有, g和p要传输给客户端, 而客户端会生成b这1个随机数, 通过DH算法客户端和服务端可以算出同样的密钥. 在这过程中a和b并不参与网络传输, 安全性大大提升. 因为p和g是大数, 所以即使在网络传输中p, g, A, B都被劫持, 靠现在的计算力算力也无法破解.

  

- 连接迁移

  TCP连接基于四元组(源IP, 源端口, 目的IP, 目的端口), 切换网络时至少会有一个因素发生变化, 导致连接发送变化. 当连接发送变化是, 如果还是用原来的TCP连接, 则会导致连接失败, 就得等到原来的连接超时后重新建立连接, 所以我们有时候发现切换到一个新的网络时, 即使网络状况良好, 但是内容还是需要加载很久. 如果实现的好, 当检测到网络变化时, 立即建立新的TCP连接, 即使这样, 建立新的连接还是需要几百毫秒时间.

  QUIC不受四元组的影响, 当这四个元素发生变化时, 原连接依然维持. 原理如下:QUIC不以四元素作为表示, 而是使用一个64位的随机数, 这个随机数被称为Connection ID, 即使IP或者端口发生变化, 只要Connection ID没有变化, 那么连接依然可以维持.

  

- 队头阻塞/多路复用

  HTTP/1.1和HTTP/2都存在队头阻塞的问题(Head Of Line blocking).

  TCP是个面向连接的协议, 即发送请求后需要收到ACK消息, 以确认对象已接受数据. 如果每次请求都要在收到上次请求的ACK消息后再请求, 那么效率无疑很低. 后来HTTP/1.1提出了Pipeline技术, 允许一个TCP连接同时发送多个请求. 这样就提升了传输效率.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7m7ia8NsHT3SLlBSVYkiawMO4iaPt1uKCQpgSCF9WmjULhljgrjGibWTAgA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  在这样的背景下, 队头阻塞发生了. 比如, 一个TCP连接同时传输10个请求, 其中1,2,3个请求给客户端接收, 但是第四个请求丢失, 那么后面第5-10个请求都被阻塞. 需要等第四个请求处理完毕后才能被处理. 这样就浪费了带宽资源.

  因此, HTTP一般又允许每个主机建立6个TCP连接, 这样可以更加充分的利用带宽资源, 但每个连接中队头阻塞的问题还是存在的.

  HTTP/2的多路复用解决了上述的队头阻塞问题. 在HTTP/2中, 每个请求都被拆分为多个`Frame`通过一条TCP连接同时被传输, 这样即使一个请求被阻塞, 也不会影响其他的请求.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7NTRRQoowE6VzJOw5LftWM62XWMIyVOiabGjH3c3Fp5ic4genATGSqlCA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  但是, HTTP/2虽然可以解决`请求`这一粒度下的阻塞, 但`HTTP/2`的基础TCP协议本身却也存在队头阻塞的问题. HTTP/2的每个请求都会被拆分成多个Frame, 不同请求的Frame组合成Stream, Stream是TCP上的逻辑传输单元, 这样HTTP/2就达到了一条连接同时发送多个请求的目标, 其中`Stram1`已经正确送达, Stram2中的第三个Frame丢失, TCP处理数据是有严格的前后顺序, 先发送的Frame要先被处理, 这样就会要求发送方重新发送第三个Frame, Steam3和Steam4虽然已到达但却不能被处理, 那么这时整条链路都会被阻塞.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7WElXLdyA18badJfNGmCBbFMrcxWicnnhhB2HjcDawH10yFhtBhQlYRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  不仅如此, 由于HTTP/2必须使用HTTPS, 而HTTPS使用TLS协议也存在队头阻塞问题. TLS基于Record组织数据, 将一对数据放在一起加密, 加密完成后又拆分成多个TCP包传输. 一般每个Record 16K, 包含12个TCP包, 这样如果12个TCP包中有任何一个包丢失, 那么整个Record都无法解密.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7uyfhIatwGicJfB2PtXyKeZPe3raxE73U6bxibhMpkd8M4yeB1KUODmVA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  队头阻塞会导致HTTP/2在更容易丢包的弱网络环境下比HTTP/1.1更慢.

  QUIC是如何解决队头阻塞的问题的? 主要有两点:

  - QUIC的传输单位是Packet, 加密单元也是Packet, 整个加密, 传输, 解密都基于Packet, 这就能避免TLS的阻塞问题.
  - QUIC基于UDP, UDP的数据包在接收端没有处理顺序, 即使中间丢失一个包, 也不会阻塞整条连接. 其他的资源会被正常处理.

- 拥塞控制

  拥塞控制的目的是避免过多的数据一下子涌入网络, 导致网络超出最大负荷. QUIC的拥塞控制与TCP类似, 并在此基础上做了改进. 先来看看TCP的拥塞控制.

  - 慢启动: 发送方像接收方发送一个单位的数据, 收到确认后发送2个单位, 然后是4个, 8个依次指数增长, 这个过程中不断试探网络的拥塞程度.
  - 避免拥塞: 指数增长到某个限制之后, 指数增长变为线性增长
  - 快速重传: 发送方每一次发送都会设置一个超时计时器, 超时后认为丢失, 需要重发
  - 快速恢复: 在上面快速重传的基础上, 发送方重新发送数据时, 也会启动一个超时定时器, 如果收到确认消息则进入拥塞避免阶段, 如果仍然超时, 则回到慢启动阶段.

  QUIC重新实现了TCP协议中的Cubic算法进行拥塞控制, 下面是QUIC改进的拥塞控制的特性:

  - 热插拔

    TCP中如果要修改拥塞控制策略, 需要在系统层面今次那个操作, QUIC修改拥塞控制策略只需要在应用层操作, 并且QUIC会根据不同的网络环境, 用户来动态选择拥塞控制算法.

  - 前向纠错 FEC

    QUIC使用前向纠错(FEC, Forword Error Correction)技术增加协议的容错性. 一段数据被切分为10个包后, 一次对每个包进行异或运算, 运算结果会作为FEC包与数据包一起被传输, 如果传输过程中有一个数据包丢失, 那么就可以根据剩余9个包以及FEC包推算出丢失的那个包的数据, 这样就大大增加了协议的容错性.

    这是符合现阶段网络传输技术的一种方案, 现阶段带宽已经不是网络传输的瓶颈, 往返时间才是, 所以新的网络传输协议可以适当增加数据冗余, 减少重传操作.

    ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7GUtEXbia2ARDs2Trj7R89xeZibicZR0ibqQuOPfcUV4VtiblERFx8wiceKQQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  - 单调递增的Packer Number

    TCP为了保证可靠性, 使用`Sequence Number`和ACK来确认消息是否有序到达, 但这样的设计存在缺陷.

    超时发生后客户端发起重传, 后来接受到了ACK确认消息, 但因为原始请求和重传请求接受到的ACK消息一样, 所以客户端就不知道这个ACK对应的是原始请求还是重传请求. 这就会造成歧义.

    - RTT: Round Trip Time, 往返事件
    - RTO: Retransmission Timeout, 超时重传时间

    如果客户端认为是重传的ACK, 但实际上是右图的情形, 会导致RTT偏小, 反之会导致RTT偏大.

    ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7RIcBibHuSPdicwVmQAddbLRILu2lqhHfDCiboX0ZoNdCBIutp5Wmpqj7g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

    QUCI解决了上面的的歧义问题, 与`Sequence Number`不同, `Packet Number`严格单调递增, 如果`Packet N`丢失了, 那么重传时Packet的标识就不会是N, 而是比N大的数字, 比如N+M, 这样发送方接收到确认消息时, 就能方便的知道ACK对应的原始请求还是重传请求.

    

  

- ACK Delay

  TCP计算RTT时没有考虑接收方接受到数据发发送方确认消息之间的延迟, 如下图所示, 这段延迟即ACK Delay. QUIC考虑了这段延迟, 使得RTT的计算更加准确.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7RHJAy83fRwCTZpibNxKv7AlQwibL0qVqXeNEC7SqBQXpmpSa9hVF9DEg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 更多的ACK块

  一般来说, 接收方收到发送方的消息后都应该发送一个ACK恢复, 表示收到了数据. 但每收到一个数据就返回一个ACK恢复实在太麻烦, 所以一般不会立即回复, 而是接受到多个数据后再回复, TCP SACK最多提供3个ACK block. 但在有些场景下, 比如下载, 只需要服务器返回数据就好, 但按照TCP的设计, 每收到三个数据包就要返回一个`ACK`, 而QUIC最多可以捎带256个`ACK block`, 在丢包率比较严重的网络下, 更多的ACK可以减少重传量, 提升网络效率.

- 流量控制

  TCP 会对每个TCP连接进行流量控制, 流量控制的意思是让发送方不要发送太快, 要让接收方来得及接受, 不然会导致数据溢出而丢失, TCP的流量控制主要通过滑动窗口来实现的. 可以看到, 拥塞控制主要是控制发送方的发送策略, 但没有考虑接收方的接收能力, 流量控制是对部分能力的不起.

  QUIC只需要建立一条连接, 在这条连接上同时传输多条`Stream`, 好比有一条道路, 量都分别有一个仓库, 道路中有很多车辆运送物资. QUIC的流量控制有两个级别: 连接级别(Connection Level)和Stream 级别(Stream Level).

  对于单条的Stream的流量控制: Stream还没传输数据时, 接收窗口(flow control recevice window)就是最大接收窗口, 随着接收方接收到数据后, 接收窗口不断缩小. 在接收到的数据中, 有的数据已被处理, 而有的数据还没来得及处理. 如下图, 蓝色块表示已处理数据, 黄色块表示违背处理数据, 这部分数据的到来, 使得Stream的接收窗口缩小.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7z8z3EzbB2s1uBJjntKicWULKH4huaaiak9JfczW3l7iaiaiaur0DmGvDsdw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  随着数据不断被处理, 接收方就有能力处理更多数据. 当满足`(flow control receivce offset - consumed bytes) < (max receive window/2)`时, 接收方会发送`WINDOW_UPDATE frame`告诉发送方你可以再多发送数据, 这时候`flow control receive offset`就会偏移, 接收窗口增大, 发送方可以发送更多数据到接收方.

  ![图片](https://mmbiz.qpic.cn/mmbiz_png/sXiaukvjR0RAulVD5jBffHcxF0qlSEDia7Ly7ovz96FzdicpkYS6rTv1J6DwfZHumJ0L5hYia3lqcjB1P5vOhoh4EA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  Stream级别对防止接收端接收过多数据作用有限, 更需要借助`Connection`级别的流量控制. 理解了`Stream`流量那么也很好理解`Connection`的流控. Stream中,

  ```
  接收窗口=最大接受窗口 - 已接收数据
  ```

  而对于Connection来说:

  ```
  接收窗口 = Stream1 接收窗口 + Stream2 接收窗口 + ... + StreamN 接收窗口
  ```

  
