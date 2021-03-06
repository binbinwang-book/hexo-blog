---
title: FPS与Jank卡顿指标
date: 2021-11-21 20:51
tags: [FPS,Jank,卡顿]
categories: [技术方案]
---

# FPS与Jank卡顿指标

近期在跟进处理 客户端FPS自动化监控 的功能，学习了客户端原生获取FPS接口，以及使用测试脚本获取手机FPS的方法（Perfdog工具），不过这篇文章打算聊一个更通用的话题，这个话题可以用下面这个问题概括：

```
FPS低一定代表卡顿吗？FPS高一定表示不卡顿吗？如何衡量卡顿率才是最准确的？
```

如果你很清楚上面这个问题的答案，那就可以跳过这篇文章了。

本篇文章按照如下的目录进行陈述，表达内容从iOS设备角度出发。

```
一、为什么会卡顿
二、FPS指标能不能衡量卡顿率
三、如何构建精准的卡顿衡量指标
```

## 一、为什么会卡顿

卡顿的本质是掉帧，意图搞清楚为什么会卡顿，也即探索为什么会掉帧。

Tips：下面这段需要具备一定程度CPU/GPU交互的基础知识，可点击阅读这篇文章[How CPU and GPU Work Together](https://www.omnisci.com/technical-glossary/cpu-vs-gpu)，该文章中的视频示例非常生动形象。

iOS 设备显示器每绘制完一帧画面，复位时就会发送一个 VSync (垂直同步信号) ，并且此时切换帧缓冲区 (iOS 设备是双缓存+垂直同步)；在读取经 GPU 渲染完成的帧缓冲区数据进行绘制的同时，还会通过 CADisplayLink 等机制通知 APP 内部可以提交结果到另一个空闲的帧缓冲区了；接着 CPU 就开始计算 APP 布局，计算完成交由 GPU 渲染，渲染完成提交到帧缓冲区；当 VSync 再一次到来的时候，切换帧缓冲区……

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwmvzk044zj30rs0g0q3g.jpg)

按时序描述绘制流程就是：
>> 0. iOS 设备完成上一帧画面的绘制，电子枪恢复到原位
>> 1. 显示器会发出一个垂直同步信号（VSync），并切换帧缓冲区
>> 2. 视频控制器按VSync信号逐帧读取帧缓冲区中由GPU渲染完成的内容，经过数据转换后交给显示器，触发电子枪渲染画面
>> 3. CPU 通过 CADisplayLink等机制（双缓冲区切换刷新机制，不同平台不同）通知App可以提交结果到另外一个空白的帧缓冲区
>> 4. CPU 开始计算App布局，计算完成交由GPU渲染
>> 5. GPU 渲染计算完成后交由帧缓冲区，接着回到步骤0，继续重复

当 VSync 到来准备切换帧缓冲区时，若空闲的帧缓存区并未收到来自 GPU 的提交，此次切换就会作罢，设备显示系统会放弃此次绘制，从而引起掉帧。

由此可知，不管是 CPU 还是 GPU 哪一个出现问题导致不能及时的提交渲染结果到帧缓冲区，都会导致掉帧。优化界面流畅程度，实际上就是减少掉帧（iOS设备上大致是 60 FPS），也就是减小 CPU 和 GPU 的压力提高性能。

## 二、FPS指标能不能衡量卡顿率

FPS 全称是Frames Per Second，即：每秒传输帧数。

在上述显示器绘制流程中，正常情况iOS平台1秒钟要将上述环节执行60次，当每秒钟执行次数降低到30以下时，人眼就能明显看出屏幕的卡顿（即不流畅），可能影响执行次数的原因也即是该环节中的各个参与方，比如：
- 信号到来时 CPU 没有计算完（比如：大量计算）
- 信号到来时 GPU 没有渲染完（比如：异步绘制）

**但FPS为50帧一定表示不卡顿吗？**

不一定，比如：FPS为50帧，前200ms渲染一帧，后800ms渲染49帧，虽然帧率50，但依然觉得非常卡顿。

**FPS为20帧一定表示卡顿吗？**

也不一定，有些平台没有渲染任务时（可以理解为屏幕静止），均匀FPS为调低到15帧。

所以我们可以初步得到一个结论：

FPS高绝大部分场景下是不卡顿的，但仅用FPS高就得出不卡顿的结论，是错误的。

我们要对FPS进行加工处理，辅助一些其它的指标和逻辑，来构建卡顿率监测指标。

## 三、如何构建精准的卡顿衡量指标（Jank算法）

### （一）FrameTime

在开始介绍 Jank卡顿算法前，我们先介绍一个名词：FrameTime.

FrameTime的定义是：两帧画面间隔耗时（也可简单认为是单帧渲染耗时），我们这里引用同学内部测试集成工具PerfDog的一张图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwn1fx7nmhj30u00bo75l.jpg)

我们着重看图中的"真实帧耗时"，它指的是：屏幕真实刷新时，两帧之间的耗时。

拿我们上面举的一个例子说明：前200ms渲染一帧，后800ms渲染49帧。

那么第一帧的FrameTime就是 200ms，后49帧的平均FrameTime就是16.32(800/49) ms，这足以帮助我们发现第一帧出现卡顿的情况。

### （二）Google的Jank指标

Google 提出的 Jank指标很简单，主要采用的就是 FrameTime，只要发生一次未更新，就算触发一次 Jank：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwn1twvtxkj30u00bggme.jpg)

如果我们使用Google 提出的 Jank指标 用于我们平日的卡顿统计，大家可以想一下会发生什么问题？

大概率会频繁触发 Jank上报，多到让我们根本无法使用这个指标。

为什么？

以iOS为例，屏幕稳定刷新率为60fps，按Google的Jank指标，在快速滑动过程中，fps降低到50实际上用户基本是无感知的，但会命中Jank指标，因为fps降低必然意味着FrameTime有所延长。

所以 Google 提出的 Jank 指标可以算是：精准卡顿率，内部开发时可以使用该指标建立监控，只要Jank在一定可接受范围内是允许上线的。

除了发布前的监控，我们还想监控现网的用户卡顿情况，一方面我们不希望用户频繁上报，另外一方面我们又希望不要错过用户真正遇到卡顿的情况，那么我们需要进一步改造一下Jank指标。

### （三）人体工程与卡顿

很多时候并不是帧率低就代表一定卡顿，卡顿毕竟是人主观的感受，所以这里一定程度上还和人体工程有关系。

- A选项：持续25帧率的视频
- B选项：80%是60帧率，20%是25帧率的视频

B选项的平均帧率远大于A选项，但实际观看过程中，反而是B选项会让人明显感受到卡顿，所以这里涉及到"视觉惯性"的问题。

视觉预期帧率，用户潜意识里认为下帧也应该是当前帧率刷新比如一直60帧，用户潜意识里认为下帧也应该是60帧率。刷新一直是25帧，用户潜意识里认为下帧也应该是25帧率。但是刷新如果是60帧一下跳变为25帧，扰乱用户视觉惯性。这个时候就会出现用户体验的卡顿感。

此外，即使保持稳定帧率也不能过低，人眼所能分辨画面不连续的临界点在 24帧，当画面刷新频率低于24fps，即使画面再稳定，观赏时也会感受到不连续性。

### （四）PerfDog团队Jank优化指标

这里以腾讯PerfDog工具制定的Jank优化指标为例，指标规则是：

同时满足两条件，则认为是一次卡顿Jank.

- ①Display FrameTime>前三帧平均耗时2倍。
- ②Display FrameTime>两帧电影帧耗时 (1000ms/24*2≈83.33ms)。


同时满足两条件，则认为是一次严重卡顿BigJank.

- ①Display FrameTime >前三帧平均耗时2倍。
- ②Display FrameTime >三帧电影帧耗时(1000ms/24*3=125ms)。

制定如此规则的依据就是人体工程中的"视觉惯性"和"连贯临界"，通过这样的规则我们可以更精准地收集到用户卡顿情况，更高效地帮助我们分析问题：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwn28o2d8yj30u00b03zi.jpg)

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)