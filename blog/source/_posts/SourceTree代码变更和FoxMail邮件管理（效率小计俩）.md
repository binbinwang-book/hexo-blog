---
title: SourceTree代码变更和FoxMail邮件管理（效率小计俩）
date: 2022-03-19 17:27
tags: [SourceTree,Foxmail]
categories: [奇技淫巧]
---

# SourceTree代码变更和FoxMail邮件管理（效率小计俩）

## 代码变更溯源

工作时，我们经常会想要查看一个类文件的变更历史，最常见的场景是："卧槽，谁改了我的代码"

新版本的Xcode溯源自我感觉相当难用，所以这里我们介绍一个工具 SourceTree 来完成这项工作。

1. 将项目工程加载到 SourceTree

当我们把项目工程拖到 SourceTree 之后，可以看到如下的内容：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0el7lt4slj21au0m8q53.jpg)

其中`BNBitcoinIndexApp`是我的项目工程名。

2. 检索文件

选择 `①文件状态` -> `②搜索文件` -> `③查看选中的修改日志`

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0el49wcs6j21270j4jv8.jpg)

3. 查看文件变更

如此可以看到所有改动到该文件的commit（是按时间顺序排列）。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0el4tmujzj20ys0mjdiw.jpg)

## 邮件管理

公司会发几十封、多的时候上百封邮件，邮箱被各种我们不关心的邮件塞满，渐渐地有价值的邮件也会被我们忽略。

这时我们学会对邮件进行管理，邮件管理分为两个步骤：首先是对邮件进行分类，其次是对邮件进行规则过滤。

### 邮件组

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ellbm8fxj206i042q2w.jpg)

点击 Foxmail -> 新建文件夹，假设你对专利比较感兴趣，可以新建文件夹`专利相关`：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0elmtq10zj206y0bxt8w.jpg)

### 邮件过滤

这里我们使用 Foxmail 自带的 规则匹配：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0elpdju2pj20a8082dgc.jpg)

任意右键点击一封右键，然后选择 `创建规则`，创建如下的规则：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0elq2ky2wj20iy0bet8z.jpg)

点击`应用到历史邮件`，稍等加载一会我们的邮件分组就完成了：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0elr1vnggj20cw05ut8p.jpg)

--------------
文章首发：[问我社区](http://www.wenwoha.com/blog_detail-1324.html)

**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)