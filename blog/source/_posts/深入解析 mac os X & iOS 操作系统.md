---
title: 深入解析 mac os X & iOS 操作系统
date: 2021-08-23 03:02
tags: [MacOS,BSD,Mach,Derwin]
categories: [计算机原理]
---

# 深入解析 mac os X & iOS 操作系统

## 问题

1. Mac OS 系统是直接通过 CPU 操作 控制器 读写，还是依赖了 DMA ？

2. 把内存存入磁盘已经是 存储管理器 层面做的事情了，iOS 还能手动操作把内存存入磁盘吗？

3. iOS 能把缓存期间的数据存到磁盘里吗？

4. mac OS 系统使用了虚拟内存技术吗？ 是选用哪种方案实现的？

5. iOS 的 App 可以使用共享内存通信吗？

6. mac OS进程的调度算法是什么？

7. macOS平台的线程是用户态还是内核态的？调度算法是什么样的？

8. 从操作系统的角度解释一下为什么iOS UI刷新要放在主线程？

## 第4章 Mach-O格式、进程以及线程内幕

Mach-O 为Mach Object 文件格式的缩写，它是一种用于可执行文件、目标代码、动态库、内核转储的文件格式。

### 4.3 通用二进制格式（Universal Binary）

通用二进制 只不过 是将支持的各种架构的二进制文件进行打包,所以叫"胖二进制"反而更好理解。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtpzt3cy16j60ob03cq3d02.jpg)

因为Mach-O的特性，所以我们可以把打包 framework 时可以将 模拟器 和 真机 的framework 进行合并。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtpzul5i7gj60k10abwf402.jpg)

通用二进制文件里包含多个 Mach-O 在各个架构下的二进制代码。

#### 4.3.1 Mach-O 二进制格式

Mach-O 是 OS X维护的自己独有的二进制格式，具有一个固定的文件头<mach-o/loader.h>

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtpzwh1ogmj60ny0a2js302.jpg)

### 4.4 动态库

#### 4.4.1 启动时库的加载

Mach-O 镜像中有很多"空洞"，即对外部的库和符号的引用，这些空洞要在程序启动时填补。

通常情况下使用的是 dyld 作为动态链接器，从字面上看，就是"填补空洞"，也就是说，查找进程中所有的符号和库的依赖关系，然后解决这些关系。

#### 4.4.2 共享库缓存

我们在开发时，经常会听到 iOS 的动态库的概念，可以实现使用的时候再被加载到内存中。

那么从操作系统层面上，动态库加载机制 是如何实现的呢？

动态库加载机制使用了共享库缓存技术，指的是一些库经过预先链接，然后保存在磁盘上的一个文件中。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)