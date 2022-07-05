---
title: iOS UI视图总篇
date: 2021-07-24 13:17
tags: [iOS视图]
categories: [iOS]
---

# iOS UI视图总篇

这篇文章介绍 UI视图 相关的内容，所涉及的知识点目录如下：

```
一、UITableView 相关

二、事件传递 & 视图响应

三、图像显示原理

四、卡顿 & 掉帧

五、绘制原理 & 异步绘制

六、离屏渲染
```

## 一、UITableView 相关

### （一）重用机制

cell = [tableView dequeueReusableCellWithIdentifier:Identifier]

真正被重用的时候，才会调用 [cell prepareForReuse] 方法，cell 消失并不会调用 prepareForReuse 。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkikwao2jj30j608imy7.jpg)

代码实现重用原理，基本原理是通过两个 NSMutableSet：

- 重用池
- 正在使用的

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkinm2ls8j30md0agtgb.jpg)

如果确定一个 cell 是一定会被创建的，那么可以提前对 Identifier 进行 registerClass 的操作，这样通过 dequeueReusableCellWithIdentifier: 取出来的 cell 一定是不为 nil 的。

### (二)数据源同步问题

在实际开发过程中，tableView 数据源的同步是一大令人头疼的问题，追加数据、变更数据、删除数据都有可能导致 tableView crash，所以处理好 数据源同步问题 ，是掌握 tableView 的关键。

一个最常见的crash问题是 数据源变更 没有放到 主线程，导致 tableView UI 刷新时， 数据源已经不是触发 UI 刷新时的数据，导致数据不匹配 crash。

那么数据源同步有哪两种方案呢？

#### 并发访问、数据拷贝

以下面这个 case 为例：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkiumy7baj30na0comyo.jpg)

大致解释一下这里的情况：

子线程正在进行网络请求操作，比如正在上传一个视频，但上传过程中用户把这个视频给删除了，因为上传是异步的，我们基本不可能在清理上传数据时，取消掉上传的网络请求。

如果这时网络回包上传成功了，那么就会把数据抛给主线程，重新刷新 UI ，这样就会把刚刚我们实际上已经删除的数据又加回来了。

上面这个问题在处理发表流程中非常常见，是一个基本必须处理的坑，那么这个问题要怎么处理呢？大家看一下下面的流程图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkiyu2dooj30nd0cg0v1.jpg)

清楚怎么处理了吗？

就是在用户删除本地数据的时，把这个数据拷贝一份塞到 deleteArray 中，等到网络请求回包后，这个数据能不能展示，需要经过 deleteArray 的过滤。

#### 串行访问

串行的思路很简单，就是在用户手动处理数据变更时，必须等到子线程操作完成，才能进行下一步操作。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkj1204r0j30n60d6mz4.jpg)

#### 对比

并行方案 和 串行方案 有着不同的应用场景，一般来说：

- 耗时比较长的异步操作，可以采用 并行方案 ，比如： CDN上传；
- 耗时比较短的异步操作，可以采用 串行方案 ，比如： cgi访问。

## 二、事件传递 & 视图响应

### （一）UIView 和 CALayer 的关系与区别

UIView = CALayer + 响应链

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkot8175zj30ns059dh6.jpg)

展示布局实际上是 CALayer.contents 决定的，contents对应一个位图。

你可能会说 UIView 也有 BackgroundColor 属性，那又是为什么呢？

实际上 UIView.backgroundColor == CALayer.backgroundColor ,也就是说 UIView.backgroundColor 是对 CALayer.backgroundColor 同名属性方法的一个封装。

>> 通过单一设计原则，明确了 UIView 和 CALayer 的分工。

### （二）事件传递

谁来响应手势，和我喊了一个人的名字，谁来响应是一个事情：

首先是某一篇区域的人听到我喊了这个人的名字，然后是这个人自己来回应我。

所以手势响应需要两个步骤：明确手势的响应区间 >> 决定哪个元素来响应。

所以也就对应着两个方法：

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;

- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event;

那这里实际上存在一个问题：pointInside: withEvent: 响应顺序是如何呢？

类比到我喊人的名字，但有8个区域，那这8个区域按照什么样的顺序判断是否响应我呢?

>> 1. 通过加入到视图上的顺序，「倒序」遍历视图的 hitTest:withEvent: 的方法
>> 2. 如果 hitTest:withEvent: 返回的是 nil ，那么就按顺序继续「倒序」遍历，直到没有可响应的对象为止。

那你可能好奇了，你说的这个顺序中并没有说到 pointInside:withEvent: ，它又有什么用呢？

hitTest:withEvent: 底层会调用 pointInside:withEvent: 方法，判断手势在不在控件上，如果你已经优先实现了 hitTest:withEvent: ，那么就不会再调用 pointInside:withEvent: 方法了，可以理解为一个控件只需要实现其中一个方法，就能达到自定义事件传递的效果。

#### hitTest:withEvent: 系统实现流程

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkq5izl5oj30nf0pd0ww.jpg)

#### 视图事件响应

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkr8ky7vcj30i0045wfz.jpg)

#### 知识延伸：UIApplication 和 UIApplicationDelegate 的关系

main() 函数中执行了 UIApplicationMain() 函数：

```
int main(int argc, char * argv[]) { 
	@autoreleasepool { 
		return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class])); 
	} 
}
```

那 UIApplication 有什么用呢？ 

UIApplication 是应用程序的象征，每一个 App 都有自己的 UIApplication 对象，而且是单例：

通过[UIApplication sharedApplication]可以获得这个单例对象，这也是 iOS 程序启动后创建的第一个对象。

通过 UIApplication 对象，我们可以进行一些系统级别的操作：

- applicationIconBadgeNumber：设置应用程序图标右上角的红色提醒数字

那 UIApplication 和 UIApplicationDelegate 之间有什么关系呢？

1.UIApplication 是 App 和 系统 之间通信的一个接口

2.所有 ViewController 默认遵守了 UIApplicationDelegate

3.一旦有系统级别的消息要通知当前页面，通过 UIApplication 通知它的 delegate 对象，让 delegate 对象来响应这些系统事件，包括：

- 应用程序的生命周期事件（如程序启动和关闭）
- 系统事件（如来电）
- 内存警告
- 接收到远程通知
- 从其它应用程序跳入

## 三、图像显示原理

### 渲染路线

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkrawu6j7j30hj08wjt3.jpg)

1. CPU 和 GPU 两个硬件是通过总线连接起来的，CPU将计算后的「bitmap」经过「总线」输出给「GPU」

2. GPU拿到「bitmap」后，进行图层渲染，将生成的结果放到「frame buffer」中

3. 最后由硬件控制器根据 sync 信号，在指令之前去 「frame buffer」提取显示内容，最终显示到屏幕上。

举例：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkrdw1my4j30hn04tq3q.jpg)

绘制好的内容最终会通过 Core Animation 框架 提供给 GPU OpenGL 渲染管线。

### CPU 在渲染中承担的工作

1. 布局

- UI布局（frame设置、label文字size计算等）
- 文本计算

2. 绘制

- 调用 drawRect: 方法进行绘制，利用Quartz 2D提供的API绘制图形

3. 编解码

- 比如 setImage: 解码图片

4. 提交位图

### GPU 在渲染中承担的工作

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkrl9srhij30hf07sq3q.jpg)


## 四、卡顿 & 掉帧

### （一）UI 卡顿背后的原因

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkroi0x9oj30hi07mq44.jpg)

60fps，也即60帧，也即16.7ms就要产生一帧动画，也就是在这 16.7ms 内CPU和GPU协同产出一帧数据，如果没有按时产出，那就会导致掉帧。

为什么是60fps ? 这是因为人眼与大脑之间的协作无法感知超过60fps的画面更新

### （二）滑动优化方案

#### 1.从CPU角度进行优化

- 对象创建、销毁
- 预排版（布局计算、文本计算）
- 预渲染（文本等异步绘制、图片编解码等）

把不必要的工作放到子线程去做，让 CPU 有更多精力和时间去响应用户的交互。

#### 2.从GPU角度进行优化

- 纹理渲染
- 视图混合

## 五、绘制原理 & 异步绘制

### (一)UIView 的绘制原理

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkrtz4pm0j30i208l0u1.jpg)

### （二）异步绘制

关键方法：displayLayer

[layer.delegate displayLayer:]

- 代理负责生成对应的 bitmap
- 设置该 bitmap 作为 layer.contens 属性的值

![](https://tva1.sinaimg.cn/large/008i3skNgy1gqkrvzerrlj30hu09hwfs.jpg)

## 六、离屏渲染

### （一）什么是离屏渲染，如何解释离屏渲染？

当我们设置某一些 UI 视频时，如果指令是在「未全部合成」之前不能展示的，那么就触发了「离屏渲染」，常见的比如：圆角（当和maskToBounds一起使用）、蒙层、遮罩等，

离屏渲染 的概念起源于 GPU 层面，指的是：GPU在当前屏幕缓冲区以外新开辟了一个缓冲区进行渲染操作。

### （二）为什么要避免离屏渲染？

触发离屏渲染时，毫无疑问会增加GPU的工作量，而增加GPU的工作量，可能会导致 CPU+GPU 生成单个帧的总工作时间加起来超过 16.7ms， 会导致掉帧或卡顿。而且：

- 离屏渲染 创建了新的渲染缓冲区，会造成内存更多开销
- 多通道渲染管线，最终还需要把多通道管线进行合成，这会增加 GPU 的负担。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)