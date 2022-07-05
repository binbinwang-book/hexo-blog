---
title: 使用 Hexo & Github 搭建博客
date: 2021-07-11 21:14
tags: 大前端
categories: [大前端]
---

## 使用 Hexo & Github 搭建博客

一直想搭建属于自己的博客，用来记录自己的所学。

但都因为各种各样奇怪的时间安排搁置了，导致我甚至开始用起了公司的git工具记录知识点.. 

今天终于把博客搭建完毕，顺便也把搭建博客路上遇到的坑记录下来。

### 先备知识

#### （一）使用 Hexo & Github 搭建博客的原因

首先，我们思考一下，相比直接把内容发在 简书/掘金 平台上，自建博客有什么优势？

- 1. 通过自己的域名可以访问博客，不用担心平台跑路/维护风险
- 2. 可以自己选择博客样式
- 3. 自己的博客相当于自己的一份名片，通过优化SEO可以做到很好的扩大影响力的效果。

现在搭建博客可以有N多工具，比如：买腾讯云服务器搭建博客、使用Jelly配置又或者自己自建网站。

但以上流程对于只是想拥有一份记录自己所学的博客需求来说，时间成本和支出成本都是偏高的。

Hexo 可以让我们便捷地搭建博客，使用他人已经设计好的博客模板；github的pages功能可以为我们提供免费的服务器，让我们零成本搭建属于自己的博客。

便捷、低成本、扩展性好 就是采用 Hexo & Github 的主要原因。

#### （二）username/nickname On Github

要区分要 github 上 username 和 nickname 的区别，这点在搭建博客时非常重要

#### （三）配置 Hexo

要配置好 Hexo ，此外了解一定的 nodeJs 基础可以让你更清楚你在配置过程中都在做什么，而不是只跟着教程操作而已，比如 npm install 是在做什么、以及为什么 package.json 可以很便捷地关联库的原因等等。

#### （四）命令行操作工具

有一定的命令行操作基础，能灵活使用 iTerm ，熟悉 git 语法更佳。

#### （五）区分 brew、npm、pip、apt-get 指令。

这四个指令实际上是 `不同系统下管理软件包的指令`。

```
一、brew
即Homebrew，是Mac OSX上的软件包管理工具，能在Mac中方便的安装软件或者卸载软件， 只需要一个命令。默认都是安装到brew的指定目录“/usr/local/Cellar”下，然后在“/usr/local/bin”下创建对应的软连接来使用的。如果安装多个不同版本的库，可以修改对应的软连接就可以了

二、npm
（全称Node Package Manager，即“node包管理器”）是Node.js預設的、用JavaScript編寫的軟體套件管理系統。

三、pip
python软件包管理系统，可以利用它安装python包，默认都安装到当前python版本的⁨python3.7⁩/site-packages⁩文件夹下

四、​apt-get
linux命令，适用于deb包管理式的操作系统，主要用于自动从互联网的软件仓库中搜索、安装、升级、卸载软件或操作系统
```

#### （六）了解 hexo 指令的含义

比如 hexo init 、hexo d -g 、hexo s。

### 搭建博客流程

#### （一）使用 github pages 创建简易博客

##### 1. 在github建立博客仓库

首先在自己的 github 上创建一个仓库，命名为 ： username.github.io 

Tips:这里的指的是你自己github帐号的username，比如我的username就是 BNineCoding，我创建的仓库就是：https://github.com/BNineCoding/bninecoding.github.io

##### 2. 为自己的仓库选择page theme

创建完成后，点击 Setting - Pages - choose Theme

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsdc2vkmx3j31c80lfq7a.jpg)

可以任意选一个 theme ，这种默认的博客样式是github为我们提供的，选完 theme 之后，你就可以通过域名 username.github.io 访问属于你自己的博客了，比如点击可查看我的博客地址：

[BNine的博客（点击查看）](https://bninecoding.github.io/)

#### （二）使用 Hexo 选择博客模板

当你完成 github pages 配置后，你可能会非常兴奋，以为自己的博客已经建成了。实际上 github pages 提供的只是一个简单的页面，如果想做成真正类似网页的博客样式，我们需要使用 Hexo 。

##### 1. Hexo init

在自己的桌面上创建一个新的文件夹，比如我命名为 blog，使用命令行工具 cd blog 文件夹后，执行： hexo init

然后就会看到 blog 文件夹中产生了一系列的文件：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsdccpqthbj310q0bcmzr.jpg)

接着我们通过执行 hexo d -g、hexo server -p 5000 两个命令将博客进行部署、本地预览，

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsdf7d7sinj30ww0u0ned.jpg)

可以看到博客已经部署在了 http://localhost:5000 网站上，我们使用浏览器访问即可看到博客页面：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsdcgjvqs7j30vy0u0gv9.jpg)

但接下来我们要思考的问题是，怎么做才能把这里的博客和我们 github 上建立的仓库联系起来呢？答案是配置 _config.yml，这个文件是站点配置文件，承担着如下两点主要作用：

- 对外：将博客与外界进行沟通，比如 链接github、SEO
- 对内：决定博客的主要样式，配置博客架构，决定博客样式 theme

所以要将博客与github链接起来，我们要先修改 _config.yml 文件，添加如下内容：

```
deploy:
  type: git
  repository: https://github.com/xxx/xxx.github.io
  branch: main
```

上面的 `repository ` 要填的就是你在第一步创建的 github 博客仓库。

配置完毕后，我们再来对博客进行部署，执行： hexo d -g，然后就可以看到我们的 github 仓库里已经有了刚刚通过 hexo init 创建的这些文件，也就是说部署时会将文件推到我们通过 repository 确认的仓库。

接着等待几分钟，我们就可以通过 xxx.github.com 访问到部署后的博客样式了（注意！github网站博客更新不是实时的，很大可能你刚部署完访问链接还是旧的博客样式，可以等几分钟或者多刷新几次）。

##### 2. 主题theme配置

如果你进行到了这一步，可能不满足于通过 Hexo init 提供的默认 博客样式 ，这时我们就可以在 github 上选择自己喜欢的 theme 样式，以我的博客选择的theme样式为例：

[hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md)

关于 `theme` 的配置，你所选的 `theme` 的 `github` 基本都有一份十分详尽地配置描述，如果你选的`theme`主题配置不够详尽，那建议你还是不要选它。

![matery主题配置文档截图](https://tva1.sinaimg.cn/large/008i3skNgy1gsdcqq8d6hj30u00wfjy0.jpg)

### 博客进阶配置

我们记录的博文如果能被搜索引擎收录，但对知识地传播会更加有效，我们也会有更大的动力去写更好的文章，那就需要对自己博客站点进行 SEO 配置优化，已经有前人说得非常好，感兴趣可以访问如下链接：

[hexo博客的高级SEO优化](https://zhuanlan.zhihu.com/p/344927945)

[Hexo 博客添加百度sitemap](https://www.jianshu.com/p/ab44b916a8b6)

[Hexo博客提交百度收录SEO](https://zhuanlan.zhihu.com/p/128033054)



------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)