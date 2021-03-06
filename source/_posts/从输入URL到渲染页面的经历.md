---
title: 从输入URL到渲染页面的经历
date: 2018-10-03 09:28:19
tags: [技术, TCP/IP, HTTP headers, 面试]
---

昨天百词斩视频一面，从经典问题**URL 从输入到展示网页的过程**引发出了一系列血案。。。直接从自信问到自卑，而且是体无完肤支离破碎那种。。。开始后悔**为啥不复习《计算机网络》**~

<!--more-->

### first blood —— ==DNS 原理==

**Q1: DNS 解析域名的过程？**

域名到 IP 地址的解析是有分布在因特网上的最多**域名服务器程序**共同完成的。域名服务程序在专设的站点上运行，而人们也常常把运行域名域名服务器程序的机器称为**域名服务器**。

当某个进程需要把域名解析为 IP 地址的时候，就会调用**解析程序**，并称谓 DNS 的一个客户，把待解析的域名放在 DNS 请求报文中，以 UDP 用户数据报方式发给本地域名服务器（在本地域名服务器，可以随意添加域名，甚至把 baidu.com 解析到本地 IP）。本地域名服务器查找到域名后，把对应的 IP 地址放在回答报文中返回，如果本地域名服务器不能回答该请求，则此域名服务器就成为 DNS 的另一个客户，并向其他域名服务器发出查询请求，直到找到能回答该请求的域名服务器为止。

![DNS解析过程.png](https://i.loli.net/2018/09/29/5baee1ee00f6d.png)

**Q2: 是否缓存？缓存在哪？**

会有缓存，缓存到本地文件中。

windows 下查看 dns 缓存命令：`ipconfig /displaydns`;

![DNS缓存](https://ws1.sinaimg.cn/large/0064OUUqly1fvq7b9altkj30ki0gd75m.jpg)

### double kill —— ==TCP 深入==

**Q1: 为什么握手需要三次？**

首先，三次握手的过程处理：

1. 第一次握手：建立连接时，客户端发送 syn 包（syn=j）到服务器，并进入 SYN_SENT 状态，等待服务器确认；
2. 第二次握手：服务器收到 syn 包，必须确认客户的 SYN（ack=j+1），同时自己也发送一个 SYN 包（syn=k），即 SYN+ACK 包，此时服务器进入 SYN_RECV 状态
3. 第三次握手：客户端收到服务器的 SYN+ACK 包，向服务器发送确认包 ACK(ack=k+1），此包发送完毕，客户端和服务器进入 ESTABLISHED（TCP 连接成功）状态，完成三次握手

![三次握手](https://ws1.sinaimg.cn/large/0064OUUqly1fvuqnbriatj30ob0grao7.jpg)

三次握手的必要性：第一次是客户端向服务端发出建立连接的请求，第二次是服务端接收到请求，并询问客户端是否正常连接，第三次是客户端响应服务端的询问，并告知服务端可以正常连接，然后可以传输数据。所谓必要性，就是保证建立连接的双方保证彼此都能正常收发信息以及一次请求发送。

**两次握手不行吗？**

两次握手的情况就会导致，至少其中某一方没有收到对方的确认信息，然后不知道是否可以建立连接。

**四次握手会怎么样？**

三次握手已经能保证通信双方正常连接发送数据，每多一次握手就会导致多一次性能浪费。

#### 通俗解释版本（知乎）

```
三次握手：
“喂，你听得到吗？”
“我听得到呀，你听得到我吗？”
“我能听到你，今天balabala……”

两次握手：
“喂，你听得到吗？”
“我听得到呀”
“喂喂，你听得到吗？”
“草，我听得到呀！！！！”
“你TM能不能听到我讲话啊！！喂！”
“……”

四次握手：
“喂，你听得到吗？”
“我听得到呀，你听得到我吗？”
“我能听到你，你能听到我吗？”
“……不想跟傻逼说话”

作者：匿名用户
链接：https://www.zhihu.com/question/24853633/answer/114872771
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

**Q2: 四次挥手的作用和原理过程**

1. 先由客户端向服务器端发送一个 FIN，请求关闭数据传输。
2. 当服务器接收到客户端的 FIN 时，向客户端发送一个 ACK，其中 ack 的值等于 FIN+SEQ
3. 然后服务器向客户端发送一个 FIN，告诉客户端应用程序关闭
4. 当客户端收到服务器端的 FIN 是，回复一个 ACK 给服务器端。其中 ack 的值等于 FIN+SEQ

![四次挥手](https://ws1.sinaimg.cn/large/0064OUUqly1fvuqmp6oxpj30o60i27l1.jpg)

当被动方收到主动方的 FIN 报文通知时，它仅仅表示主动方没有数据再发送给被动方了。但未必被动方所有的数据都完整的发送给了主动方，所以被动方不会马上关闭 SOCKET,它可能还需要发送一些数据给主动方后，再发送 FIN 报文给主动方，告诉主动方同意关闭连接，所以这里的 ACK 报文和 FIN 报文多数情况下都是分开发送的。

**PS:** 连接的断开请求不一定发生在客户端。

**Q3: 保证可靠传输的原因是什么？**

> **超时重传机制**：发送方发送的报文中含有序列号，每当发送一个报文后，就启动一个**计时器（RTO）**，该计时器的时间一般是有当前网络来决定的，一个 RTT 指的是当一个报文从发送到接收到对应的 ACK 标志的时间，RTO 的决定一般是发送方尝试发送几个报文，然后取平均 RTT 时间来决定计时器的值。 当发送一个报文以后，发送方在计时范围以内，如果**没有接收到**相应的 ACK 确认报文，那么发送方就会重传该报文。

> **快速重传机制**：该机制指的是，发送方一直发送报文，不会每发一次报文就都要等待到这个报文的 ACK 标志才发送下个报文。 当接收方发送接受的序列号不对的时候，发送连续的 3 个 ACK 标志，告诉发送方，这个报文在传输过程中出现了丢包。发送方如果接收到某个相同序列号的三个 ACK 报文，那么此时立马重发该报文，不用等待计时器的时间结束。

### Triple Kill —— ==聊聊 IP 协议（网际协议）==

**Q: 你对协议了解的多吗？还想问一下 IP 协议**

IP 是给因特网的每一台联网设备规定一个地址。

通常与 IP 协议配套使用的还有三个协议：

- 地址解析协议 ARP
- 网际控制报文协议 ICMP
- 网际组管理协议 IGMP

![IP数据报](https://ws1.sinaimg.cn/large/0064OUUqly1fvurcdajnmj30nq0eo77k.jpg)

### Quadr Kill —— ==HTTP header==

**Q1: 有哪些 header, 描述作用**

1.  request Headers

    - Accept
    - Accept-Charset
    - Cache-Control
    - Connection
    - Date
    - Host
    - Referer
    - User-Agent
    - Method
    - ……

2.  response Headers
    - Accept-Ranges
    - Age
    - Allow
    - Expires
    - ……

[MDN HTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)

**Q2: 服务端是怎么控制 cookie 的**

```Java
Cookie cookie = new Cookie(name, value);
//禁止客户端js获取cookie
cookie.setHttpOnly(true);
//是否允许浏览器仅通过HTTPS连接传回cookie
cookie.setSecure(false);
//设置路径，这个路径即该工程下都可以访问该cookie 如果不设置路径，那么只有设置该cookie路径及其子路径可以访问
cookie.setPath("/");
//cookie.setMaxAge(360000);
```

### Penta Kill + 越塔强杀 —— ==HTTPS==

**HTTPS 最后为什么要协商的是对称加密秘钥，换成非对称加密不行吗？**

这个问题，最后特意问了一下面试官，得到回复是：因为如果到了协商最终加密算法的阶段，就认为已经是安全的，所以需要一个效率高的加密算法。而非对称加密的算法解密比对称加密的效率要低(又是效率，，`innerHTML`那个就是效率问题)。
