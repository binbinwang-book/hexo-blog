---
title: 个人博客SEO踩坑记录
date: 2021-07-24 14:39
tags: [个人博客SEO,百度收录,Google收录,DNS]
categories: [大前端]
---

# 个人博客SEO踩坑记录

自我买完域名后，一直想把个人博客接入 百度收录/Google 收录，中间踩了几个坑导致这事隔了好几天才接完，把坑记录如下。

## 一、github Pages域名配置

刚买的域名在 github 上配置 pages 时会进行如下的提示：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss2ch6iohj30ri0f3myq.jpg)

这是因为我们刚买的域名没有配置 DNS解析记录 导致的，前往 腾讯云控制台 ，然后添加如下的 DNS解析记录：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss2pkzayvj30oi08b3yx.jpg)

这里配置 DNS解析记录 时有个小点要同步下：如果你的blog CNAME 配置的域名是没有 www 开头的，那么在配置DNS解析记录这里，主机记录选择 @ 就好了。

如果你主机记录选择了 www ，但 CNAME 文件目录配置却没有输入 www，比如只输入了 bninecoding.com，而不是 www.bninecoding.com ，那么在你下一次更新博客后，你的配置就失效了。关于「A」和「CNAME」的区别，如下图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss7fflydtj30ld0dstac.jpg)

所以这里讲究的是域名使用的一致性， 「DNS解析记录」、「github pages 自定义域名」、「博客工程CNAME配置域名」、「SEO站点域名配置」等，都要保持一致才不会出问题。

添加完毕后我们差不多要等待半小时（我一般等10分钟就够了）让DNS解析生效，返回 github pages 重新配置 自定义域名即可：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss2jqslpqj30k00cxab6.jpg)

按上述配置后我们的博客已经可以打开了，但在更新文档后，github pages 又会报warning：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss3484oltj30qw0eugn6.jpg)

如果只是为了博客运转，这个warning无视就好了，但为了一次性解决完配置问题，我又给 DNS解析记录 添加了一条：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss35sit08j30ry09bt96.jpg)

其中的ip地址是我自己博客的ip地址，比如我的是 www.bninecoding.github.io，查询出对应的 ip地址 进行配置即可：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss36h44suj30mj0abaar.jpg)

如此一来 CNAME、A记录 我们都加上了，彻底解决了配置问题。[吐血]

## 二、百度收录

百度收录在验证域名时会让你选择校验方式，类似下面这样：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss1dzsls1j30rx0oe0ud.jpg)

经过我亲自测试，最简单的方法还是「CNAME验证」：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss0wr3droj30kl0h3q3v.jpg)

因为我买的是腾讯云，所以需要在腾讯云控制台添加记录，要注意的是区分清楚「主机记录」和「记录值」，填错了就肯定解析不了：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss0y55ud8j30oh04uglv.jpg)

验证完毕之后，使用 baidu_url_submitter 做推送时，需要输入推送的 token ，推送的token可以在百度站长查看到，位置如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss0zt1mg6j30w90mqgo9.jpg)

如此在部署博客时，就可以愉快地把文章同步到百度收录了，同步后的截图如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss10o2v6aj31cy0ga437.jpg)

## 三、Google收录

踩过百度收录的坑之后，再搞Google收录相对就简单点。

打开收录网站：

[Google收录网站](https://search.google.com/search-console)

填写完毕后，同样需要做一下域名校验，用来说明你添加的域名是你自己的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss15a5uv1j30nw06jdg0.jpg)

校验完毕后就可以看到自己的Google收录域名控制台了：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss15xpgpmj31a20u0ace.jpg)

然后我们去检索我们的博客有没有被Google收录：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss16ef0jmj30na0j2dhv.jpg)

可以看到，Google收录秒过，不得不说大Google这效率和体验真的是杠杠的。

## 四、DNS相关

配置博客的时候，经常会听到要进行DNS校验的事情，尽管大学里计网里学过这个知识，但这里真实应用接触了，特把知识点记录如下。

### 1. 什么是DNS

dns==domain name server（域名服务器）

域名系统 (DNS) 将人类可读的域名 (例如，www.amazon.com) 转换为机器可读的 IP 地址 (例如，192.0.2.44)。

### 2. DNS基础知识

Internet 上的所有计算机，从您的智能手机或笔记本电脑到可提供大量零售网站内容的服务器，均通过使用编号寻找另一方并相互通信。这些编号称为 IP 地址。当您打开 Web 浏览器并前往一个网站时，您不必记住和输入长编号。而是输入域名 (入 example.com)，然后在正确的位置结束。

Amazon Route 53 等 DNS 服务是一种全球分布式服务，它将人类可读的名称 (如 www.example.com) 转换为数字 IP 地址 (如 192.0.2.1)，供计算机用于相互连接。Internet 的 DNS 系统的工作原理和电话簿相似，都是管理名称和数字之间的映射关系。DNS 服务器可以将名称请求转换为 IP 地址，从而控制最终用户在 Web 浏览器中输入域名时所访问的服务器。这些请求称为查询。

### 3. DNS 服务的类型

授权型 DNS：一种授权型 DNS 服务提供一种更新机制，供开发人员用于管理其公用 DNS 名称。然后，它响应 DNS 查询，将域名转换为 IP 地址，以便计算机可以相互通信。授权型 DNS 对域有最终授权且负责提供递归型 DNS 服务器对 IP 地址信息的响应。Amazon Route 53 是一种授权型 DNS 系统。

递归型 DNS：客户端通常不会对授权型 DNS 服务直接进行查询。而是通常连接到称为解析程序的其他类型 DNS 服务，或递归型 DNS服务。递归型 DNS 服务就像是旅馆的门童：尽管没有任何自身的 DNS 记录，但是可充当代表您获得 DNS 信息的中间程序。如果递归型 DNS 拥有已缓存或存储一段时间的 DNS 参考，那么它会通过提供源或 IP 信息来响应 DNS 查询。如果没有，则它会将查询传递到一个或多个授权型 DNS 服务器以查找信息。

### 4. DNS 如何将流量路由到您的 Web 应用程序？

![](https://tva1.sinaimg.cn/large/008i3skNgy1gss1qw59nbj30wc0oqn0n.jpg)

1. 用户打开 Web 浏览器，在地址栏中输入 www.example.com，然后按 Enter 键。

2. www.example.com 的请求被路由到 DNS 解析程序，这一般由用户的 Internet 服务提供商 (ISP) 进行管理，例如有线 Internet 服务提供商、DSL 宽带提供商或公司网络。

3. ISP 的 DNS 解析程序将 www.example.com 的请求转发到 DNS 根名称服务器。

4. ISP 的 DNS 解析程序再次转发 www.example.com 的请求，这次转发到 .com 域的一个 TLD 名称服务器。.com 域的名称服务器使用与 example.com 域相关的四个 Amazon Route 53 名称服务器的名称来响应该请求。

5. ISP 的 DNS 解析程序选择一个 Amazon Route 53 名称服务器，并将 www.example.com 的请求转发到该名称服务器。

6. Amazon Route 53 名称服务器在 example.com 托管区域中查找 www.example.com 记录，获得相关值，例如，Web 服务器的 IP 地址 (192.0.2.44)，并将 IP 地址返回至 DNS 解析程序。

7. ISP 的 DNS 解析程序最终获得用户需要的 IP 地址。解析程序将此值返回至 Web 浏览器。DNS 解析程序还会将 example.com 的 IP 地址缓存 (存储) 您指定的时长，以便它能够在下次有人浏览 example.com 时更快地作出响应。有关更多信息，请参阅存活期 (TTL)。

8. Web 浏览器将 www.example.com 的请求发送到从 DNS 解析程序中获得的 IP 地址。这是您的内容所处位置，例如，在 Amazon EC2 实例中或配置为网站终端节点的 Amazon S3 存储桶中运行的 Web 服务器。

9. 192.0.2.44 上的 Web 服务器或其他资源将 www.example.com 的 Web 页面返回到 Web 浏览器，且 Web 浏览器会显示该页面。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)
