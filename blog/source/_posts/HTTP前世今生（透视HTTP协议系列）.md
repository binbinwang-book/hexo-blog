---
title: HTTP前世今生（透视HTTP协议系列）
date: 2022-01-10 02:46
tags: [HTTP]
categories: [计算机网络]
---

# HTTP前世今生（透视HTTP协议系列）

1989 年，任职于欧洲核子研究中心（CERN）的蒂姆·伯纳斯 - 李（Tim Berners-Lee）发表了一篇论文，提出了在互联网上构建超链接文档系统的构想。这篇论文中他确立了三项关键技术。

- URI：即统一资源标识符，作为互联网上资源的唯一身份；
- HTML：即超文本标记语言，描述超文本文档；
- HTTP：即超文本传输协议，用来传输超文本。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gy7jzzyw5jj309n0723yh.jpg)

这三项技术在如今的我们看来已经是稀松平常，但在当时却是了不得的大发明。基于它们，就可以把超文本系统完美地运行在互联网上，让各地的人们能够自由地共享信息，蒂姆把这个系统称为“万维网”（World Wide Web），也就是我们现在所熟知的 Web。

所以在这一年，我们的英雄“HTTP”诞生了，从此开始了它伟大的征途。

## 一、URI和URL的区别

统一资源标志符URI就是在某一规则下能把一个资源独一无二地标识出来。 拿人做例子，假设这个世界上所有人的名字都不能重复，那么名字就是URI的一个实例，通过名字这个字符串就可以标识出唯一的一个人。 现实当中名字当然是会重复的，所以身份证号才是URI，通过身份证号能让我们能且仅能确定一个人。 那统一资源定位符URL是什么呢。也拿人做例子然后跟HTTP的URL做类比，就可以有：

动物住址协议://地球/中国/浙江省/杭州市/西湖区/某大学/14号宿舍楼/525号寝/张三.人

可以看到，这个字符串同样标识出了唯一的一个人，起到了URI的作用，所以URL是URI的子集。URL是以描述人的位置来唯一确定一个人的。 在上文我们用身份证号也可以唯一确定一个人。对于这个在杭州的张三，我们也可以用：

身份证号：123456789

来标识他。 所以不论是用定位的方式还是用编号的方式，我们都可以唯一确定一个人，都是URl的一种实现，而URL就是用定位的方式实现的URI。

- URI ：Uniform Resource Identifier，统一资源标识符；
- URL：Uniform Resource Locator，统一资源定位符； 
- URN：Uniform Resource Name，统一资源名称。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gy7jvn1mkvj30m80deaac.jpg)

URL是一种具体的URI，它不仅唯一标识资源，而且还提供了定位该资源的信息。

URI是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位。

URI：统一资源标识 URL：统一资源定位 URN：统一资源名称

例如： www.baidu.com是URL. www.baidu.com/index.html 是URL 同时也是URI。 所以，URL 就是 URI 的 定位。

但 URI 不一定是 URL。 因为 URI有一类子集是 URN，它是命名资源 但不指定如何定位资源。 如： mailto 需要 加上 相应的结构参数，才能进行 统一资源定位。 如： mailto: xxxxx@qq.com

因此，三者之间的关系是： URI 一定是 URL URN + URL 就是 URI。

## 二、HTTP演化关键里程碑

### （一）HTTP 1 - 浏览器大战

1995 年，网景的 Netscape Navigator 和微软的 Internet Explorer 开始了著名的“浏览器大战”，都希望在互联网上占据主导地位。

### （二）HTTP 2 - Google浏览器Chrome

Google 首先开发了自己的浏览器 Chrome，然后推出了新的 SPDY 协议，并在 Chrome 里应用于自家的服务器，如同十多年前的网景与微软一样，从实际的用户方来“倒逼”HTTP 协议的变革，这也开启了第二次的“浏览器大战”。

历史再次重演，不过这次的胜利者是 Google，Chrome 目前的全球的占有率超过了 60%。“挟用户以号令天下”，Google 借此顺势把 SPDY 推上了标准的宝座，互联网标准化组织以 SPDY 为基础开始制定新版本的 HTTP 协议，最终在 2015 年发布了 HTTP/2，RFC 编号 7540。

### （三）HTTP 3 - QUIC协议

在 HTTP/2 还处于草案之时，Google 又发明了一个新的协议，叫做 QUIC，而且还是相同的“套路”，继续在 Chrome 和自家服务器里试验着“玩”，依托它的庞大用户量和数据量，持续地推动 QUIC 协议成为互联网上的“既成事实”。

“功夫不负有心人”，当然也是因为 QUIC 确实自身素质过硬。

在去年，也就是 2018 年，互联网标准化组织 IETF 提议将“HTTP over QUIC”更名为“HTTP/3”并获得批准，HTTP/3 正式进入了标准化制订阶段，也许两三年后就会正式发布，到时候我们很可能会跳过 HTTP/2 直接进入 HTTP/3。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)