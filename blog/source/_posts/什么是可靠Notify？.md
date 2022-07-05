---
title: 什么是可靠Notify？
date: 2022-02-19 16:24
tags: [Notify]
categories: [计算机网络]
---

# 什么是可靠Notify？

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gziwbmjzwtj20px0dtt9w.jpg)

这套业内常用的「Notify/NewSync」机制却存在一些问题

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gziwc3o4h7j20sy0eb3zd.jpg)

当有新的红点产生时，后台同步提升 「finder sequence」 和 「wx sequence」，触发「Notify通道」，

如果「客户端长连接」还在，那么会及时去「收红点」，

若「长连不存在」，则会在重连时，借助「NewSync」告知用户下发红点。

但这会导致两个问题：「长连时」，因「调用和后台更新时序」不确定，可能会重复两次拉取，「Notify一次」，「New Sync」一次。

且这种方案依赖了「消息服务wx sequence」，如果同步不当，会对消息服务产生影响。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1gziwcokzs2j20ti0dl3ze.jpg)

所以「客户端和后台」一起将红点sync接入了「可靠Notify」的方案，

后台提升「finder sequence」后，走到「可靠Notify」层级，

将「Nofity信令」以kv的形式存储在「墓碑」中，

如果客户端在线，则触发「finderSync」收红点时，同时返回ack告知mmProxy，然后后台再从墓碑中清理
Notify信令。

重连mmProxy首先「检索墓碑」，查看是否有未消费的Nofity。

可靠Notify约减少20%的红点请求量。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)