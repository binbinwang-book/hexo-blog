---
title: iOS 动画（从API调用到微机原理）
date: 2021-08-29 20：45
tags: [动画,CPU,GPU,CoreAnimation,CoreGraphics,YYAsyncLayer,YYFPSLabel,AsyncDisplayKit,pop,卡顿,优化]
categories: [iOS]
---

# iOS 动画（从API调用到微机原理）

工作以来写了不少关于 iOS动画 的需求，今天在写新的动画demo时，突然觉得是时候把 iOS 动画相关的内容做一个全面总结了，于是花了一天的时间整理完了这篇文章。

本文目录如下：

```
# 一、iOS 动画应用API全解
## （一）UIView 实现动画效果
## （二）CoreAnimation 实现动画效果
## （三）UIViewPropertyAnimator 交互动画
## （四）自定义View
## （五）自定义动画

# 二、动画原理篇
## （一）先备知识
## （二）主线程和UI绘制的关系
## （三）绘制图像过程中CPU和GPU的分工（垂直同步信号、二级缓存等）
## （四）图像绘制原理

# 三、开源库源码分析
## （一）YYAsyncLayer
## （二）YYFPSLabel
## （三）AsyncDisplayKit
## （四）pop动画库

# 四、性能优化篇
## （一）CPU 和 GPU 监控
## （二）UI 卡顿一套优化流程

# 五、其它
## （一）开源项目的编写
## （二）对动画的总结
## （三）布局相关
## （四）提炼出的面试题目

# 六、参考文章

```

# 一、iOS 动画应用API全解

文章已分拆至:[iOS 动画应用API全解](https://bninecoding.com/ios-dong-hua-ying-yong-api-quan-jie.html)

# 二、动画原理篇

文章已分拆至:[iOS 动画原理篇](https://bninecoding.com/ios-dong-hua-yuan-li-pian.html)

# 三、开源库源码分析

文章已分拆至:[iOS 动画 开源库源码分析](https://bninecoding.com/ios-dong-hua-kai-yuan-ku-yuan-ma-fen-xi.html)

# 四、性能优化篇

## （一）CPU 和 GPU 监控

稍后更新。

## （二）UI 卡顿一套优化流程

稍后更新。

# 五、其它

## （一）开源项目的编写

## （二）对动画的总结

## （三）布局相关

### 1. 动画执行过程中 autoresize是否生效， autoresize 的原理

### 2. yoga的使用

## （四）提炼出的面试题目

# 六、参考文章

```
动画应用篇：
- iOS动画篇：UIView动画:  https://www.jianshu.com/p/5abc038e4d94
- iOS动画篇：核心动画: https://www.jianshu.com/p/d05d19f70bac
- iOS动画篇：自定义View:  https://www.jianshu.com/p/9ac974756f77
- iOS动画篇：自定义动画: https://www.jianshu.com/p/673585164bd2
- iOS动画篇：下拉刷新动画: https://www.jianshu.com/p/3c51e4896632

- 位图在两个环境下的不同概念
http://jartto.wang/2018/12/09/bitmap/

- UIViewPropertyAnimator 的应用：
https://swift.gg/2017/04/20/quick-guide-animations-with-uiviewpropertyanimator/
https://juejin.cn/post/6844903760225943566

- 概念- 绘图和动画 CoreGraphics/CoreAnimation/CoreText/CoreImage/Layer
https://www.jianshu.com/p/404da65539a6
https://www.jianshu.com/p/e3a3aedefd23

- 隐式动画：https://www.jianshu.com/p/043e7ec7f3ef

- 逐帧动画 和 关键帧动画
https://www.jianshu.com/p/13c231b76594

- 为什么必须在主线程操作UI
https://juejin.cn/post/6844903763011076110

- 图像渲染原理（二级缓存、垂直同步信号）
https://cloud.tencent.com/developer/article/1638099
https://zhuanlan.zhihu.com/p/307909741
http://chuquan.me/2018/09/25/ios-graphics-render-principle/

- render server 
https://www.jianshu.com/p/d95eb13b531c

- CPU 和 GPU 之间的通信：
https://zhuanlan.zhihu.com/p/91959702
http://chuquan.me/2018/08/26/graphics-rending-principle-gpu/

- YYAsyncLayer:
https://cloud.tencent.com/developer/article/1175151

- YYFPSLabel:
https://github.com/yehot/YYFPSLabel


- CPU 和 GPU 数据监控
https://segmentfault.com/a/1190000004164291
https://xcoder.tips/gpu-utilization/

- Pop 的使用
https://www.jianshu.com/p/671ef9342db2
https://www.jianshu.com/p/59ba5510cc26
```

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)