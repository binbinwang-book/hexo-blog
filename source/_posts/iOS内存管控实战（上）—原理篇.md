---
title: iOS内存管控实战（上）—原理篇
date: 2022-06-05 22:30
tags: [iOS,内存]
categories: [iOS]
---

# iOS内存管控实战（上）—原理篇

因文章单篇过长，按照 原理、分析工具 和 实战 拆分成上、中、下三部分，点击阅读。
- [iOS内存管控实战（上）—原理篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-shang-yuan-li-pian.html)
- [iOS内存管控实战（中）-分析工具篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-zhong-fen-xi-gong-ju-pian.html)
- [iOS内存管控实战（下）—实战篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-xia-shi-zhan-pian.html)

## 前言

近期有个同事遇到了内存泄露导致卡死的问题，具体原因是：view A监听了某个事件，但viewA本身因为block使用不当，出现了泄露问题，用户多次操作导致viewA泄露了72次。当事件发出通知时，72个被泄露的viewA同时去触发某个操作，导致手机卡死。这个问题挺经典的，平时如果不注意 block weakify/strongify ，内存泄露真出现时，影响颇大。

加上有同事在debug内存泄露时秀了一把 Xcode memory Debug，让我觉得是时候将过去所学的内存管理相关的知识进行一番汇总，本篇文章更多的工作在于汇总和黏连各个优秀的文章，如果您有精力，建议还是阅读文章末尾粘贴的原文链接，在此也感谢前辈们的分享。

本篇文章目录为：

- 一、iOS内存基础原理
	- （一）iOS系统内存上限
	- （二）内存泄露的三种类型
	- （三）Clean & Dirty & Compressed 内存
- 二、内存分析工具
	- （一）分析工具一览
	- （二）Xcode memory debugger 和 指令操作
	- （三）Facebook循环引用检测框架：FBRetainCycleDetector
	- （四）轻量级的内存泄露检测框架：MLeaksFinder
- 三、内存优化实战
	- （一）图片压缩优化

## 一、iOS内存基础原理

### （一）iOS系统内存上限

在 stack overflow 上，有人对单个 App 能够使用的最大内存做了统计：iOS app max memory budget。以 iPhone XS Max 为例，总共的可用内存是 3735 MB（比硬件大小小一些，因为系统本身也会消耗一部分内存），而单个 App 可用内存达到 2039 MB，达到了 55%。当 App 使用的内存超过这个临界值，就会发生 OOM 崩溃。可以看出，单个 App 的可用物理内存实际上还是很大的，要发生 OOM 崩溃，绝大多数情况下都是程序本身出了问题。

OOM常见原因：

- 内存泄漏：最常见的原因之一就是内存泄漏。
- UIWebview 缺陷：无论是打开网页，还是执行一段简单的 js 代码，UIWebView 都会占用大量内存，同时旧版本的 css 动画也会导致大量问题，所以最好使用 WKWebView。
- 大图片、大视图：缩放、绘制分辨率高的大图片，播放 gif 图，以及渲染本身 size 过大的视图（例如超长的 TextView）等，都会占用大量内存，轻则造成卡顿，重则可能在解析、渲染的过程中发生 OOM。

实战说明：最近我在优化图片发表的逻辑，debug发现在选择多张超高清大图进行压缩时，内存占用甚至会飙升到2G，按照 stack overflow 上的说法，2G已经触碰OOM红线了，所以我对图片压缩的逻辑进行了优化，感兴趣的可以到我[博客](https://bninecoding.com/)查看最新的一篇文章。

### （二）内存泄露的三种类型

先看看 Leaks，从苹果的开发者文档里可以看到，一个 app 的内存分三类：

- ❎Leaked memory: Memory unreferenced by your application that cannot be used again or freed (also detectable by using the Leaks instrument).
- ‼️Abandoned memory: Memory still referenced by your application that has no useful purpose.
- ✅Cached memory: Memory still referenced by your application that might be used again for better performance.

其中 Leaked memory 和 Abandoned memory 都属于应该释放而没释放的内存，都是内存泄露，而 Leaks 工具只负责检测 Leaked memory，而不管 Abandoned memory。在 MRC 时代 Leaked memory 很常见，因为很容易忘了调用 release，但在 ARC 时代更常见的内存泄露是循环引用导致的 Abandoned memory，Leaks 工具查不出这类内存泄露，应用有限。

### （三）Clean & Dirty & Compressed 内存

#### 1. Pages Memory

内存是由系统管理，一般以页为单位来划分。在 iOS 上，每一页包含 16KB 的空间。一段数据可能会占用多页内存，所占用页总数乘以每页空间得到的就是这段数据使用的总内存。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xcx3lt20j21080h9wf9.jpg)

内存页按照各自的分配和使用状态，可以被分为 Clean 和 Dirty 两类。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xcxlyh7qj21bk0b8aav.jpg)

以上面的代码为例，申请一块长度为 80000 字节的内存空间，按照一页 16KB 来计算，就需要 6 页内存来存储。

- 当这些内存页开辟出来的时候，它们都是 Clean 的
- 当向处于第一页的内存写入数据时，第一页内存会变成 Dirty
- 当向处于最后一页的内存写入数据时，这一页也会变成 Dirty

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xcyd4byej21080860t8.jpg)

#### 2. 内存映射文件

当 App 访问一个文件时，系统内核会负责调度，将磁盘上的文件加载并映射到内存中。如果这是只读的文件，它所占用到的内存页是 Clean 的。

如下图所示，一个 50KB 的图片被加载到内存中时，需要分配 4 页内存来存储。其中第四页中有 2KB 的空间会被用来存储这个图片的数据，剩余空间可能会被用来存储其它数据。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xczc8r58j21080g63z5.jpg)

#### 3. 典型app内存类型

当内存不足的时候，系统会按照一定策略来腾出更多空间供使用，比较常见的做法是将一部分低优先级的数据挪到磁盘上，这个操作称为 Page Out。之后当再次访问到这块数据的时候，系统会负责将它重新搬回内存空间中，这个操作称为 Page In。

然而对于移动设备而言，频繁对磁盘进行IO操作会降低存储设备的寿命。从 iOS7 开始，系统开始采用压缩内存的办法来释放内存空间，被压缩的内存称为 Compressed Memory。下面依次介绍一下 iOS App 通常情况下的三种内存类型：Clean Memory 、Dirty Memory以及Compressed Memory。

#### 4. Clean Memory

Clean Memory 是指那些可以用以 Page Out 的内存，包括已被加载到内存中的文件，或者是 App 所用到的 frameworks。每个 frameworks 都有 _DATA_CONST 段，当 App 在运行时使用到了某个 framework，它所对应的 _DATA_CONST 的内存就会由 Clean 变为 Dirty。

#### 5. Dirty Memory

Dirty Memory 是指那些被 App 写入过数据的内存，包括所有堆区的对象、图像解码缓冲区，同时，类似 Clean memory，也包括 App 所用到的 frameworks。每个 framework 都会有 _DATA 段和 _DATA_DIRTY 段，它们的内存是 Dirty 的。

值得注意的是，在使用 framework 的过程中会产生 Dirty Memory，使用单例或者全局初始化方法是减少 Dirty Memory 不错的方法，因为单例一旦创建就不会销毁，全局初始化方法会在 class 加载时执行。

#### 6. Compressed Memory

当内存吃紧的时候，系统会将不使用的内存进行压缩，直到下一次访问的时候进行解压。

例如，当我们使用 Dictionary 去缓存数据的时候，假设现在已经使用了 3 页内存，当不访问的时候可能会被压缩为 1 页，再次使用到时候又会解压成 3 页。

官方对内存压缩的描述是：

```
With OS X Mavericks, Compressed Memory allows your Mac to free up memory space when you need it most. As your Mac approaches maximum memory capacity, OS X automatically compresses data from inactive apps, making more memory available.
```

大致上是在内存不够用的时候，把非活跃应用占用的内存进行压缩。可以看出相对于把dirty的内存换出到硬盘而言，这是一种折中的方案，本质上是用CPU时间换硬盘I/O时间。虽然压缩/解压会比换出/换入占用更多的CPU，但花在硬盘I/O上的时间会大大减小。

#### 7. Memory Warnings 

并非所有内存警告都是由 App 造成的，例如在内存较小的设备上，当你接听电话的时候也有可能发生内存警告。按照以往的习惯，你可能会在收到内存警告通知的时候去做一些释放内存的事情。然而内存压缩机制会使事情变得复杂。我们来看看这个例子：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdey0urvj21p40hcabl.jpg)

假设代码中的 cache 已被压缩过

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdfija93j2108089mxl.jpg)

事实上，当你尝试去再次访问 cache 对象的时候，系统会先解压这块内存

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdfslg8hj210807w0t3.jpg)

这个过程中内存使用会增加，在内存吃紧的时候，这并不是我们想要的。随后，当我们会执行大量工作去清空 cache，最终得到的内存空间和内存压缩的结果一样

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdg2q3zbj2108089mxl.jpg)

所以，相比以往的缓存手段，更加建议去调整策略，例如减少缓存使用，或者在收到内存警告的时候，将这类事情交由系统去处理。

#### 8. Caching

我们对数据进行缓存的目的是想减少 CPU 的压力，但是过多的缓存又会占用过大的内存。由于内存压缩机制的存在，我们需要根据缓存数据大小以及重算这些数据的成本，在 CPU 和内存之间进行权衡。

在一些需要缓存数据的场景下，可以考虑使用 NSCache 代替 NSDictionary，因为 NSCache 可以自动清理内存，在内存吃紧的时候会更加合理。

#### 9. 小结

通常情况下，我们所说的内存占用是指 Dirty Memory 和 Compressed Memory，Clean Memory 不需要过多关心。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdgq7ee1j20ts0ycgmu.jpg)

## 参考文章

- [WWDC 2018：iOS 内存深入研究](https://juejin.cn/post/6844903621276991502#heading-9)
- [iOS调试Block引用对象无法被释放的一个小技巧](https://cloud.tencent.com/developer/article/1508382)
- [iOS之深入解析Memory内存](https://blog.csdn.net/Forever_wj/article/details/120578784)
- [MLeaksFinder：精准 iOS 内存泄露检测工具](https://wereadteam.github.io/2016/02/22/MLeaksFinder/)
- [OS X Mavericks 中的内存压缩技术到底有多强大？](https://www.zhihu.com/question/21775223)
- [FBRetainCycleDetector + MLeaksFinder 阅读](https://www.jianshu.com/p/76250de94b93)
- [获取OC对象的所有强引用对象](https://blog.jerrychu.top/2021/01/10/%E8%8E%B7%E5%8F%96OC%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%89%80%E6%9C%89%E5%BC%BA%E5%BC%95%E7%94%A8%E5%AF%B9%E8%B1%A1/)
- [WWDC2018 图像最佳实践](https://juejin.cn/post/6844903618429059086)

——————————————

文章首发：[问我社区](http://www.wenwoha.com/)