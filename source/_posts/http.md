---
title: http1\2, https, http缓存
date: 2024-05-14 09:03:38
tags:
---

# Http1.0

浏览器与服务器只保持短暂的链接，浏览器的每次请求都需要和服务器建立一个 TCP 链接（TCP 的每次链接成本很高，都需要 client and service 三次握手和四次挥手），服务器完成请求后立即断开链接，不跟踪每个客户也不记录每次请求

# Http1.1

持久链接：引入持久链接，即默认 TCP 不关闭，可以被多个请求复用，大部分浏览器允许同时建立 6 个持久链接；
管道机制：在同一个 TCP 连接里面，client 可以发起多个请求
分块传输编码，即服务端每产生一个数据，就发送一个
新增请求方式：put\delete\options\trace\connect;
缺点：虽然允许复用，但是同一个 tcp 链接里面，所有的通信数据是按次序进行的，服务器只有处理完成一个后才会处理下一个请求，会有排队发生，产生头部阻塞

# http2：

采用二进制格式，之前版本采用的是文本格式；
完全多路复用，而非有序并阻塞的，只需要一个链接即可实现并行；解决了头部阻塞的问题
使用报头压缩，降低开销；
1、二进制协议：1.1 版本头信息是文本，（经过 ASCII 编码），数据体可以是文本，也可以是二进制，2.0 版本彻底采用二进制协议，头信息和数据体都是二进制，统称为帧，头信息帧和数据帧，解析起来更高效，错误信息更少；
2、完全多路复用，复用了 TCP 链接，在一个链接里面 client and server 同时可以发送多个请求和响应，不用按照顺序对应，避免头部阻塞；
3、报头压缩
4、服务器推送：HTTP 协议通常承载于 TCP 协议之上，在 http 协议和 tcp 协议之间增加一个安全协议层，SSL 或者 TSL，就成了 https

# https 原理

‌HTTPS 是一种应用层协议，是 HTTP 的安全版本，通过传输加密和身份认证保证了传输过程的安全性。它在 HTTP 的基础上引入了一个加密层，使用 SSL/TLS 协议来加密数据包，确保数据在传输过程中的机密性和完整性。

### HTTPS 的工作原理主要包括以下几个步骤：

客户端发送 HTTPS 请求到服务器。
服务器返回数字证书（包括公钥）给客户端。
客户端验证证书是否合法，如果合法，则生成一个随机的对称加密密钥。
客户端使用服务器的公钥将该密钥加密，然后发送给服务器。
服务器使用自己的私钥将密钥解密。
服务器和客户端使用该对称密钥进行加密和解密，完成后续的数据传输。在这个过程中，数字证书的作用是验证服务器的身份，确保客户端与服务器之间的通信安全。

## TTPS 的优势包括：

保护隐私：所有信息都是加密传播，第三方无法窃听数据。
数据完整性：一旦第三方篡改了数据，接收方会知道数据经过了篡改，保证了数据在传输过程中不被篡改。
身份认证：第三方不可能冒充身份参与通信，因为服务器配备了由证书颁发机构（CA）颁发的安全证书。
HTTPS 被广泛用于万维网上安全敏感的通讯，例如交易支付等方面，确保了网络通信的安全性和可靠性。

# 理解 Http 缓存

浏览器第一次向一个 web 服务器发起 http 请求后，服务器会返回请求的资源，并且在响应头中添加一些有关缓存的字段如：Cache-Control、Expires、Last-Modified、ETag、Date 等等。之后浏览器再向该服务器请求该资源就可以视情况使用强缓存和协商缓存。

- 强缓存：浏览器直接从本地缓存中获取数据，不与服务器进行交互。
- 协商缓存：浏览器发送请求到服务器，服务器判定是否可使用本地缓存。
- 联系与区别：两种缓存方式最终使用的都是本地缓存；前者无需与服务器交互，后者需要。
  ![](imageq.png)

## 强缓存

如图红线所示的过程代表强缓存。用户发起了一个 http 请求后，浏览器发现先本地已有所请求资源的缓存，便开始检查缓存是否过期。有两个 http 头部字段控制缓存的有效期：Expires 和 Cache-Control，浏览器是根据以下两步来判定缓存是否过期的：

查看缓存是否有 Cache-Control 的 s-maxage 或 max-age 指令，若有，则使用响应报文生成时间 Date + s-maxage/max-age 获得过期时间，再与当前时间进行对比（s-maxage 适用于多用户使用的公共缓存服务器）；
如果没有 Cache-Control 的 s-maxage 或 max-age 指令，则比较 Expires 中的过期时间与当前时间。Expires 是一个绝对时间。
注意，在 HTTP/1.1 中，当首部字段 Cache-Control 有指定 s-maxage 或 max-age 指令，比起首部字段 Expires，会优先处理 s-maxage 或 max-age。

另外下面列几个 Cache-Control 的常用指令：

no-cache：含义是不使用本地缓存，需要使用协商缓存，也就是先与服务器确认缓存是否可用。
no-store：禁用缓存。
public：表明其他用户也可使用缓存，适用于公共缓存服务器的情况。
private：表明只有特定用户才能使用缓存，适用于公共缓存服务器的情况。
经过上述两步判断后，若缓存未过期，返回状态码为 200，则直接从本地读取缓存，这就完成了整个强缓存过程；如果缓存过期，则进入协商缓存或服务器返回新资源过程。

## 协商缓存

当浏览器发现缓存过期后，缓存并不一定不能使用了，因为服务器端的资源可能仍然没有改变，所以需要与服务器协商，让服务器判断本地缓存是否还能使用。此时浏览器会判断缓存中是否有 ETag 或 Last-Modified 字段，如果没有，则发起一个 http 请求，服务器根据请求返回资源；如果有这两个字段，则在请求头中添加 If-None-Match 字段（有 ETag 字段的话添加）、If-Modified-Since 字段（有 Last-Modified 字段的话添加）。注意：如果同时发送 If-None-Match 、If-Modified-Since 字段，服务器只要比较 If-None-Match 和 ETag 的内容是否一致即可；如果内容一致，服务器认为缓存仍然可用，则返回状态码 304，浏览器直接读取本地缓存，这就完成了协商缓存的过程，也就是图中的蓝线；如果内容不一致，则视情况返回其他状态码，并返回所请求资源。下面详细解释下这个过程：

1.ETag 和 If-None-Match
二者的值都是服务器为每份资源分配的唯一标识字符串。

浏览器请求资源，服务器会在响应报文头中加入 ETag 字段。资源更新时，服务器端的 ETag 值也随之更新；
浏览器再次请求资源时，会在请求报文头中添加 If-None-Match 字段，它的值就是上次响应报文中的 ETag 的值；
服务器会比对 ETag 与 If-None-Match 的值是否一致，如果不一致，服务器则接受请求，返回更新后的资源；如果一致，表明资源未更新，则返回状态码为 304 的响应，可继续使用本地缓存，要注意的是，此时响应头会加上 ETag 字段，即使它没有变化。
2.Last-Modified 和 If-Modified-Since
二者的值都是 GMT 格式的时间字符串。

浏览器第一次向服务器请求资源后，服务器会在响应头中加上 Last-Modified 字段，表明该资源最后一次的修改时间；
浏览器再次请求该资源时，会在请求报文头中添加 If-Modified-Since 字段，它的值就是上次服务器响应报文中的 Last-Modified 的值；
服务器会比对 Last-Modified 与 If-Modified-Since 的值是否一致，如果不一致，服务器则接受请求，返回更新后的资源；如果一致，表明资源未更新，则返回状态码为 304 的响应，可继续使用本地缓存，与 ETag 不同的是：此时响应头中不会再添加 Last-Modified 字段。

<!-- https://segmentfault.com/a/1190000015816331 -->

在 HTTP 通信建立在 TCP 连接基础之上，而 TCP 连接的建立和断开需要经过“三次握手”和“四次挥手”的过程。

**一、三次握手（建立连接）**

1. 第一次握手：
   - 客户端向服务器发送一个 SYN（Synchronize Sequence Numbers，同步序列编号）报文段，该报文段中包含客户端的初始序列号（Initial Sequence Number，ISN）。此时客户端进入 SYN_SENT 状态。
   - 这个报文段的作用是告诉服务器，客户端想要建立连接，并告知服务器自己的初始序列号，以便后续的数据传输能够有序进行。
2. 第二次握手：
   - 服务器收到客户端的 SYN 报文段后，向客户端发送一个 SYN/ACK（Synchronize-Acknowledge，同步确认）报文段。
   - 这个报文段中包含服务器的初始序列号和对客户端 SYN 报文段的确认号（Acknowledgment Number，ACK）。确认号的值为客户端的序列号加一，表示服务器已经收到了客户端的 SYN 报文段，并同意建立连接。此时服务器进入 SYN_RCVD 状态。
3. 第三次握手：
   - 客户端收到服务器的 SYN/ACK 报文段后，向服务器发送一个 ACK 报文段。
   - 这个报文段中的确认号为服务器的序列号加一，表示客户端已经收到了服务器的 SYN/ACK 报文段，并确认建立连接。此时客户端和服务器都进入 ESTABLISHED 状态，连接建立成功，可以开始进行数据传输。

**二、四次挥手（断开连接）**

1. 第一次挥手：
   - 主动关闭方（通常是客户端，但也可能是服务器）发送一个 FIN（Finish，结束）报文段，用来关闭主动方到被动方的数据传送，此时主动方进入 FIN_WAIT_1 状态。
   - 这个报文段的作用是通知被动方，主动方已经没有数据要发送了，希望断开连接。
2. 第二次挥手：
   - 被动方收到主动方的 FIN 报文段后，向主动方发送一个 ACK 报文段，表示已经收到了主动方的关闭请求。此时被动方进入 CLOSE_WAIT 状态，主动方进入 FIN_WAIT_2 状态。
   - 这个 ACK 报文段只是确认收到了主动方的关闭请求，但被动方可能还有数据要发送给主动方，所以此时连接还没有完全关闭。
3. 第三次挥手：
   - 被动方发送完所有数据后，也向主动方发送一个 FIN 报文段，表示被动方也没有数据要发送了，希望断开连接。此时被动方进入 LAST_ACK 状态。
   - 这个 FIN 报文段是被动方在确认自己没有数据要发送后发出的，通知主动方可以真正地关闭连接了。
4. 第四次挥手：
   - 主动方收到被动方的 FIN 报文段后，向被动方发送一个 ACK 报文段，表示已经收到了被动方的关闭请求。此时主动方进入 TIME_WAIT 状态，被动方进入 CLOSED 状态。
   - 经过一段时间的等待（通常为 2 倍的最大报文段生存时间，即 2MSL）后，主动方也进入 CLOSED 状态，连接完全关闭。

“三次握手”和“四次挥手”的过程确保了 TCP 连接的可靠建立和安全断开，保证了数据传输的准确性和完整性。
