---
title: github push 权限被收回问题记录
date: 2021-07-29 02：35
tags: [github-push报错]
categories: [技术方案]
---

# github push 权限被收回问题记录

今天下班愉快地写博客，写完执行 hexo d -g 之后，发现报错了：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsx89bu3uoj31go0d4q63.jpg)

看了一下，关键的一句报错是：

Password authentication is temporarily disabled as part of a brownout. Please use a personal access token instead [duplicate]


去 Stack Overflow 搜了一下，是因为 github 觉得你 push 权限交给 第三方API 使用的太频繁了，不建议你通过直接暴露 github_username + github_pwd 的方式去 push 代码，建议你采用 personal access token 的方式去 push 代码。

Stack Overflow 文章：

[Password authentication is temporarily disabled as part of a brownout. Please use a personal access token instead [duplicate]](https://stackoverflow.com/questions/68191392/password-authentication-is-temporarily-disabled-as-part-of-a-brownout-please-us)

那就按着回复高赞操作一把，去github生成了自己的一把 personal access token之后，再清理掉 chainkey access 里关于 github 的记录，然后在 terminal 输入 push，就会让你输入 账号和密码。

账号继续输入github的帐号，密码就输入刚生成的token即可。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)