---
title: 短/长轮询，长连接，WebSocket
date: 2022-01-10 02:48
tags: [轮询,长连接]
categories: [计算机网络]
---

# 短/长轮询，长连接，WebSocket

## 一、短轮询

重复发送 Http 请求, 查询目标事件是否完成, 不论结果是否与上次有变化都会查询.

- 优点: 编写简单
- 缺点: 浪费带宽与服务器资源

## 二、长轮询

long poll. 每次向服务器发送请求后, 如果服务器暂时没有新数据要返回, 则服务器暂时不返回 response, 直至有新数据为止 (或者 timout 了))

- 优点: 无消息情况下不会频繁请求
- 缺点: 编写复杂

## 三、长连接

HTTP1.1 规定了默认保持长连接 (HTTP persistent connection, 也有翻译为持久连接), 数据传输完成了保持 TCP 连接不断开 (不发 RST 包, 不四次挥手),等待在同域名下继续用这个通道传输数据; 相反的就是短连接。

如果服务器没有告诉客户端超时时间也没关系, 服务端可能主动发起四次挥手断开 TCP 连接, 客户端能够知道该 TCP 连接已经无效; 另外 TCP还有心跳包来检测当前连接是否还活着, 方法很多, 避免浪费资源.

HTTP 请求时, request header 默认含有 Connection: Keep-Alive, 但是最终是否为长链接是由 server 返回的 reponse header 这些标签决定了车辆是如何运送物资的

```
telnet my.server.net 80
Trying X.X.X.X...
Connected to my.server.net.
GET /homepage.html HTTP/1.0
Connection: keep-alive
Host: my.server.net

HTTP/1.1 200 OK
Date: Thu, 03 Oct 2013 09:05:28 GMT
Server: Apache
Last-Modified: Wed, 15 Sep 2010 14:45:31 GMT
ETag: "1af210b-7b-4904d6196d8c0"
Accept-Ranges: bytes
Content-Length: 123
Vary: Accept-Encoding
Keep-Alive: timeout=15, max=100
Connection: Keep-Alive
Content-Type: text/html
```

其中的 Keep-Alive 中的 timeout=15 表示可接受的超时长度为 15 秒 (超过就会主动断开), max=100 表示可使用本 tcp 连接建立 http 的最多次数
(也就是重复利用次数)

>>> 如果不指定 timeout 时, 默认值为 60s

## 四、WebSocket

如上所述, 传统的 HTTP 是单次 request 对应单次 response, 而且每次的数据都必须由客户端发起, 服务器不会主动推送数据.

虽然 HTTP 也有”长连接”, 即 keep-alive, 但是这个”长连接”并不是真正的长连接, 其只是在一次 TCP 连接中客户端可以发送多个 request,然后获得服务端的多个 response, HTTP 的 request = response, 即一次请求一定对应一次回应, 这是不变的. 因此在这两种方案中, 每次的 request中必然要携带 header, 这样就造成了客户端与服务端处理效率的低下

于是, HTML5 时代提供了一项新的浏览器与服务器间进行全双工通讯的网络技术 - WebSocket. WebSocket 与 HTTP 类似, 同样是跑在 TCP 协议上的一种协议.
在初始建立连接时也是先经历 TCP/IP 协议的握手, 然后客户端发出建立 WebSocket 协议连接的请求, 请求建立后就会一直存在,然后在连接存在时服务端可不经过客户端请求而直接向客户端推送数据, 直至客户端与服务端的任意一端主动切断连接为止.

## 五、WebSocket 与 HTTP 的关系

WebSocket 就是为长连接而诞生的, 可以理解为 HTTP 协议的持久化补充. 在 WebSocket 第一次与服务端连接时, 发送的是 HTTP request (不过这个 request 的 header 会声明使用的是 WebSocket 协议, 并要求服务端也升级到 WebSocket 协议), 在建立连接后, 之后的交换数据都不需要再发送 HTTP request 了.

![](https://tva1.sinaimg.cn/large/008i3skNgy1gy7sag7zm8j30u00cuaav.jpg)

上面的请求与回应只会在第一次建立 WebSocket 连接时执行一次, 使用的是 HTTP 协议进行的, 在成功建立了 WebSocket 连接后, HTTP 协议也就完成任务了
(可以理解为过河拆桥), 之后就完全按照 WebSocket 协议的规定进行数据流的推送了.

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)