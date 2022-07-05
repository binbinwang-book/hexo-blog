---
title: 事件传递与响应 详解（精通iOS系列）
date: 2021-10-02 23:18
tags: [手势,事件传递,响应链]
categories: [iOS]
---

# 事件传递与响应 详解（精通iOS系列）

## 前言

一说到iOS的手势传递，可能我们脑海中都会浮现出下面三张经典的图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guz9kok56xj613l0lh40k02.jpg)


![](https://tva1.sinaimg.cn/large/008i3skNgy1guz9l61dyyj60zk0lk0u902.jpg)


![](https://tva1.sinaimg.cn/large/008i3skNgy1guz9pngu6fj60ni0q876e02.jpg)

这三张图固然十分重要，但我以过来人的经验来说明，如果只是简单理解了这三张图的流程，那么在设计复杂层级以及多window工程时，很容易设计出有问题的架构。

且在新版本中，第一和第二张图描述的响应顺序还是错误的:（，所以有必要仔细地勘探 事件传递与响应 知识点。

在开始之前我罗列了下面几个问题，如果你觉得下面几个问题你都可以对答如流，那么可以略过本篇文章。

```
1. 点击屏幕到执行业务代码，经历了什么？
2. 如何实现一个触屏反馈的效果？
3. 如何扩大所有UIView的响应范围？
4. 多层级的View响应手势时，是按什么规则进行遍历的？
5. UIResponder 和 UIView 是什么关系？
6. 如果一个 view 被加到了两个 superView 里，它的 nextResponder 是谁？
```

本篇文章会按照如下的顺序进行阐述：

- 一、什么是事件（先备知识）？
- 二、事件传递到App之前发生了什么？
- 三、事件在App内传递的三个阶段

![](https://tva1.sinaimg.cn/large/008i3skNgy1gv07y69etuj60hk0chjrv02.jpg)

## 一、什么是事件（先备知识）？

事件指的是 `UIEvent : NSObject`，它的API文档很简单：

```
typedef NS_ENUM(NSInteger, UIEventType) {
    UIEventTypeTouches,
    UIEventTypeMotion,
    UIEventTypeRemoteControl,
    UIEventTypePresses API_AVAILABLE(ios(9.0)),
    UIEventTypeScroll      API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos) = 10,
    UIEventTypeHover       API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos) = 11,
    UIEventTypeTransform   API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos) = 14,
};

typedef NS_ENUM(NSInteger, UIEventSubtype) {
    // available in iPhone OS 3.0
    UIEventSubtypeNone                              = 0,
    
    // for UIEventTypeMotion, available in iPhone OS 3.0
    UIEventSubtypeMotionShake                       = 1,
    
    // for UIEventTypeRemoteControl, available in iOS 4.0
    UIEventSubtypeRemoteControlPlay                 = 100,
    UIEventSubtypeRemoteControlPause                = 101,
    UIEventSubtypeRemoteControlStop                 = 102,
    UIEventSubtypeRemoteControlTogglePlayPause      = 103,
    UIEventSubtypeRemoteControlNextTrack            = 104,
    UIEventSubtypeRemoteControlPreviousTrack        = 105,
    UIEventSubtypeRemoteControlBeginSeekingBackward = 106,
    UIEventSubtypeRemoteControlEndSeekingBackward   = 107,
    UIEventSubtypeRemoteControlBeginSeekingForward  = 108,
    UIEventSubtypeRemoteControlEndSeekingForward    = 109,
};

/// Set of buttons pressed for the current event
/// Raw format of: 1 << (buttonNumber - 1)
/// UIEventButtonMaskPrimary = 1 << 0
typedef NS_OPTIONS(NSInteger, UIEventButtonMask) {
    UIEventButtonMaskPrimary    = 1 << 0,
    UIEventButtonMaskSecondary  = 1 << 1
} NS_SWIFT_NAME(UIEvent.ButtonMask) API_AVAILABLE(ios(13.4)) API_UNAVAILABLE(tvos, watchos);

/// Convenience initializer for a button mask where `buttonNumber` is a one-based index of the button on the input device
/// .button(1) == .primary
/// .button(2) == .secondary
UIKIT_EXTERN UIEventButtonMask UIEventButtonMaskForButtonNumber(NSInteger buttonNumber) NS_SWIFT_NAME(UIEventButtonMask.button(_:)) API_AVAILABLE(ios(13.4)) API_UNAVAILABLE(tvos, watchos);

UIKIT_EXTERN API_AVAILABLE(ios(2.0)) @interface UIEvent : NSObject

@property(nonatomic,readonly) UIEventType     type API_AVAILABLE(ios(3.0));
@property(nonatomic,readonly) UIEventSubtype  subtype API_AVAILABLE(ios(3.0));

@property(nonatomic,readonly) NSTimeInterval  timestamp;

@property (nonatomic, readonly) UIKeyModifierFlags modifierFlags API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos);
@property (nonatomic, readonly) UIEventButtonMask buttonMask API_AVAILABLE(ios(13.4)) API_UNAVAILABLE(tvos, watchos);

@property(nonatomic, readonly, nullable) NSSet <UITouch *> *allTouches;
- (nullable NSSet <UITouch *> *)touchesForWindow:(UIWindow *)window;
- (nullable NSSet <UITouch *> *)touchesForView:(UIView *)view;
- (nullable NSSet <UITouch *> *)touchesForGestureRecognizer:(UIGestureRecognizer *)gesture API_AVAILABLE(ios(3.2));

// An array of auxiliary UITouch’s for the touch events that did not get delivered for a given main touch. This also includes an auxiliary version of the main touch itself.
- (nullable NSArray <UITouch *> *)coalescedTouchesForTouch:(UITouch *)touch API_AVAILABLE(ios(9.0));

// An array of auxiliary UITouch’s for touch events that are predicted to occur for a given main touch. These predictions may not exactly match the real behavior of the touch as it moves, so they should be interpreted as an estimate.
- (nullable NSArray <UITouch *> *)predictedTouchesForTouch:(UITouch *)touch API_AVAILABLE(ios(9.0));

@end

NS_ASSUME_NONNULL_END

#else
#import <UIKitCore/UIEvent.h>
#endif
```

我们以 `UIEventType` 作为突破口。

```
typedef NS_ENUM(NSInteger, UIEventType) {
    UIEventTypeTouches,
    UIEventTypeMotion,
    UIEventTypeRemoteControl,
    UIEventTypePresses API_AVAILABLE(ios(9.0)),
    UIEventTypeScroll      API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos) = 10,
    UIEventTypeHover       API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos) = 11,
    UIEventTypeTransform   API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos) = 14,
};
```

`UIEventType` 可以帮助我们理解目前可以处理的事件的类型，截止到我写这篇文章的时间点，我看到的绝大部分的文章基本只说 `UIEventType` 有4种类型，没有提及 iOS 13.4 新增的3种类型。

通过枚举我们可以看出，截止到 2021年10月01日，`UIEventType` 一共有 7 种：

- touch events（触摸事件）
- motion events（运动事件）
- remote-control events（远程控制事件）
- press events（按压事件）
- scroll events（滚动事件，处于基本不可用的状态，Apple Doc也未给出介绍）
- hover events（手指停留在上面，依旧处于基本不可用的状态，Apple Doc也未给出介绍）
- Transform events（旋转手势，依旧处于基本不可用的状态，Apple Doc也未给出介绍）

可以说虽然`UIEventType`目前提供了7种类型，但后三种类型的介绍和使用暂时都没有被Apple重视起来，就像是不小心暴露的API，不打算继续给开发者们使用。

这也大概能解释为什么关于后三种类型的事件，网上基本没有人在讨论。一个连官方都不重视的type，开发者更不可能使用了。

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzvf5xyldj60jq0bsmxm02.jpg)

我们这里重点看前四种事件类型。

#### 1. 触摸事件

触摸事件就是我们的手指或者苹果的 Pencil（触笔）在屏幕中所引发的互动，比如轻点、长按、滑动等操作，是我们最常接触到的事件类型。触摸事件对象可以包含一个或多个触摸，并且每个触摸由 UITouch 对象表示。当触摸事件发生时，系统会将其沿着线路传递，找到适当的响应者并调用适当的方法，例如 touchedBegan:withEvent:。响应者对象会根据触摸来确定适当的方法。
触摸事件分为以下几类：

- 手势事件
	- 长按手势（UILongPressGestureRecognizer）
	- 拖动手势（UIPanGestureRecognizer）
	- 捏合手势（UIPinchGestureRecognizer）
	- 响应屏幕边缘手势（UIScreenEdgePanGestureRecognizer）
	- 轻扫手势（UISwipeGestureRecognizer）
	- 旋转手势（UIRotationGestureRecognizer）
	- 点击手势（UITapGestureRecognizer）
- 自定义手势
- 点击 button 相关

触摸事件对应的对象为 `UITouch`。

#### 2. 运动事件

iPhone 内置陀螺仪、加速器和磁力仪，可以感知手机的运动情况。iOS 提供了 Core Motion 框架来处理这些运动事件。根据这些内置硬件，运动事件大致分为三类：

- 陀螺仪相关：陀螺仪会测量设备绕 X-Y-Z 轴的自转速率、倾斜角度等。通过 Core Motion 提供的一些 API 可以获取到这些数据，并进行处理；通过系统可以通过内置陀螺仪获取设备的朝向，以此对 App UI 做出调整
- 加速器相关：设备可以通过内置加速器测量设备在 X-Y-Z 轴速度的改变；Core Motion 提供了高度计（CMAltimeter）、计步器（CMPedometer）等对象，来获取处理这些产生的数据
- 磁力仪相关：使用磁力仪可以获取当前设备的磁极、方向、经纬度等数据，这些数据多用于地图导航开发

不过官方文档中指出，这些都是属于 Core Motion 库框架，Core Motion 库中的事件直接由 Core Motion 内部进行处理，不会通过响应者链，所以 UIKit 框架能接收的事件暂时只包括摇一摇（`EventSubtype.motionShake`）。

```
- (void)motionEnded:(UIEventSubtype)motion withEvent:(UIEvent *)event {
    if (event.type == UIEventTypeMotion && event.subtype == UIEventSubtypeMotionShake) {
    	/// do something...
    }

    [super motionEnded:motion withEvent:event];
}
```

#### 3. 远程控制事件

首先不要误会，这里的远程控制并不是像黑客那样可以通过网络直接控制别人的手机，这里指的是允许响应者对象从外部附件或耳机接受命令，以便它可以管理音频和视频。目前 iOS 仅提供我们远程控制音频和视频的权限，即对音频实现暂停/播放、上一曲/下一曲、快进/快退操作。以下是它能识别的类型：

```
typedef NS_ENUM(NSInteger, UIEventSubtype) {
    // available in iPhone OS 3.0
    UIEventSubtypeNone                              = 0,
    
    // for UIEventTypeMotion, available in iPhone OS 3.0
    UIEventSubtypeMotionShake                       = 1,
    
    // for UIEventTypeRemoteControl, available in iOS 4.0
    UIEventSubtypeRemoteControlPlay                 = 100,
    UIEventSubtypeRemoteControlPause                = 101,
    UIEventSubtypeRemoteControlStop                 = 102,
    UIEventSubtypeRemoteControlTogglePlayPause      = 103,
    UIEventSubtypeRemoteControlNextTrack            = 104,
    UIEventSubtypeRemoteControlPreviousTrack        = 105,
    UIEventSubtypeRemoteControlBeginSeekingBackward = 106,
    UIEventSubtypeRemoteControlEndSeekingBackward   = 107,
    UIEventSubtypeRemoteControlBeginSeekingForward  = 108,
    UIEventSubtypeRemoteControlEndSeekingForward    = 109,
};
```

#### 4.按压事件
iOS 9.0 之后提供了 3D Touch 事件，通过使用这个功能可以做如下操作：

Quick Actions：重压 App icon 可以进行很多快捷操作
Peek and Pop：使用这个功能对文件进行预览和其他操作，可以在手机自带 “信息” 里面试验
Pressure Sensitivity：压力响应敏感，可以在备忘录中选择画笔，按压不同力度画出来的颜色深浅不一样

## 二、事件传递到App之前发生了什么？

我们一般说的事件传递的起点在于 UIApplication 所管理的事件队列中开始分发的时候，但事件真正的起点在于你手指触摸到屏幕的那一刻开始（以触摸事件为例），那么在触摸屏幕到事件队列开始分发发生了什么？我们就以一个触摸事件来说明这个过程。

- 1. 手指触摸屏幕，IOKit.framework 将事件封装成一个 IOHIDEvent 对象
- 2. 将这个对象通过 mach port（IPC 进程间通信）转发到 Springboard
- 3. Springboard 通过 mach port（IPC 进程间通信）转发给当前 App 的主线程
- 4. 前台 App 主线程的 RunLoop 接收到 Springboard 转发过来的消息之后，触发对应的 mach port 的 Source1 回调 __IOHIDEventSystemClientQueueCallback()
- 5. Source1 回调内部触发了 Source0 的回调 __UIApplicationHandleEventQueue()
- 6. Source0 回调内部，封装 IOHIDEvent 为 UIEvent
- 7. Source0 回调内部调用 UIApplication 的 +sendEvent: 方法，将 UIEvent 传给当前 UIWindow

说到这我们暂停解读几个关键词：

#### IOKit.framework

是一个系统框架的集合，用来驱动一些系统事件。IOHIDEvent 中的 HID 代表 Human Interface Device，即人机交互驱动

#### SpringBoard

是一个`应用程序`，用来`管理 iOS 的主屏幕`，除此之外像 WindowServer(窗口服务器)、bootstrapping(引导应用程序)，以及在启动时候系统的一些初始化设置都是由这个特定的应用程序负责的。它是我们 iOS 程序中，事件的第一个接收者。它只能接受少数的事件，比如：按键（锁屏/静音等）、触摸、加速、接近传感器等几种 Event，随后使用 mach port 转发给需要的 App 进程

#### Source0 和 Source1

Source0 只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。

Source1 包含了一个 mach_port 和一个回调（函数指针），被用于通过`内核和其他线程`相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程

具体可详见 [RunLoop-九里BNine](https://bninecoding.com/runloop.html)

这里相信大家应该都清楚 Runloop 和 Runtime 的区别，`Runloop`表示的是 `事件接收和分发机制`，`Runtime`表示的是`运行时决议，是patch的立足之本`，顺口一提，我们继续。

UIApplication 管理了一个事件队列，之所以是队列而不是栈，是因为队列的特点是先进先出，先产生的事件先处理。UIApplication 会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow），主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个处理过程的第一步。

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzvyw5ky3j60u019ggot02.jpg)

这部分结束，我们就聊完了当我们点击屏幕时，事件是如何传递到应用进程的。当然这里可能会有同学问到 VoiceOver 模式下，事件也是如此传递的吗？

虽然Apple没有公布过 VoiceOver 下事件传递的逻辑，但我们可以大胆猜测，在事件传递到应用前，流程是一致的。（没有单独做一套传递逻辑的必要）

## 三、事件在App内传递的三个阶段

UIWindow 接收到的事件，有的是通过响应者链传递，找到合适的响应者进行处理；有的不需要传递，直接用 first responder 来处理。这里我们主要说需要沿着响应者链传递的过程。
事件的传递大致可以分为三个阶段：

- Hit-Test（寻找合适的 view）
- Gesture Recognizer（手势识别）
- Response Chain（响应链，touch 事件传递）

通过手或触笔触摸屏幕所产生的事件，都是通过这三步去传递的，如前面提到的触摸事件和按压事件。

### （一）Hit-Test（寻找合适的 view）

其实这是确定第一响应者的过程，第一响应者也就是作为首先响应此次事件的对象。对于每次事件发生之后，系统会去找能处理这个事件的第一响应者。根据不同的事件类型，第一响应者也不同：

- 触摸事件：被触摸的那个对象
- 按压事件：被聚焦按压的那个对象
- 摇晃事件：用户或者 UIKit 指定的那个对象
- 远程事件：用户或者 UIKit 指定的那个对象
- 菜单编辑事件：用户或者 UIKit 指定的那个对象

>>> 与加速计、陀螺仪、磁力仪相关的运动事件，是不遵循响应链机制传递的。Core Motion 会将事件直接传递给你所指定的第一响应者。

#### 1. 原理

当点击一个 view，事件传递到 UIWindow 后，会去遍历 view 层级，直到找到合适的响应者来处理事件，这个过程也叫做 Hit-Test。
既然是遍历，就会有一定的顺序。系统会根据添加 view 的前后顺序，确定 view 在 subviews 中的顺序，然后根据这个顺序将视图层级转化为图层树，针对这个树，使用`倒序、深度遍历`的算法，进行遍历。之所以要倒叙，是因为最顶层的 view 最有可能成为响应者。

Hit-Test 在代码中对应的方法为：

```
func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView?
// hitTest 内部调用下面这个方法
func point(inside point: CGPoint, with event: UIEvent?) -> Bool
```

详细步骤：

- 1. keyWindow 接收到 UIApplication 传递过来的事件，首先判断自己能否接受触摸事件，如果能，那么判断触摸点在不在自己身上
- 2. 如果触摸点在 keyWindow 身上，那么 keyWindow 会从后往前遍历自己的子控件（为了寻找最合适的 view）
- 3. 遍历的每一个子控件都会重复上面的两个步骤（1.判断子控件是否能接受事件；2.触摸点在不在子控件上）
- 4. 如此循环遍历子控件，直到找到最合适的 view，如果没有更合适的子控件，那么自己就是最合适的 view

每当手指接触屏幕，UIApplication 接收到手指的事件之后，就会去调用 UIWindow 的 hitTest:withEvent:，看看当前点击的点是不是在 window 内，如果是则继续依次调用 subView 的 hitTest:withEvent: 方法，直到找到最后需要的 view。调用结束并且 hit-test view 确定之后，这个 view 和 view 上面依附的手势，都会和一个 UITouch 的对象关联起来，这个 UITouch 会作为事件传递的参数之一，我们可以看到 UITouch 的头文件中有一个 view 和 gestureRecognizers 的属性，就是 hit-test view 和它的手势。

如下图:

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzw7b4ftvj60xe0owgoh02.jpg)

补充：深度优先遍历的代码实现：

```
public class Solution { 
    private static class Node { 
        /** 
         * 节点值 
         */ 
        public int value; 
        /** 
         * 左节点 
         */ 
        public Node left; 
        /** 
         * 右节点 
         */ 
        public Node right; 
 
        public Node(int value, Node left, Node right) { 
            this.value = value; 
            this.left = left; 
            this.right = right; 
        } 
    } 
 
    public static void dfs(Node treeNode) { 
        if (treeNode == null) { 
            return; 
        } 
        // 遍历节点 
        process(treeNode) 
        // 遍历左节点 
        dfs(treeNode.left); 
        // 遍历右节点 
        dfs(treeNode.right); 
    } 
} 
```

#### 2. 实例

Hit-Test 是采用递归的方法从 view 层级的根节点开始遍历，来通过一个例子看一下它是如何工作的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzw8xx55nj60yy0gp3zm02.jpg)

UIWindow 有一个 MainView，MainView 里面有三个 subView：viewA、viewB、viewC。它们各自有两个 subView，它们的层级关系是：viewA 在最下面，viewB 在中间，viewC 最上（也就是 addSubview 的顺序，越晚 add 进去越在上面），其中 viewA 和 viewB 有一部分重叠。

如果手指在 viewB.1 和 viewA.2 重叠的方面点击，按照上面的递归方式，顺序如下图所示：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzw9fppfnj60yy0gpta402.jpg)

当点击图中位置时，会从 viewC 开始遍历，先判断点在不在 viewC 上，不在。转向 viewB，点在 viewB 上。转向 viewB.2，判断点在不在 viewB.2 上，不在。转向 viewB.1，点在 viewB.1 上，且 viewB.1 没有子视图了，那么 viewB.1 就是最合适的 view。遍历到这里也就结束了。

#### 3. 具体实现

来看一下 hitTest:withEvent: 的实现原理，UIWindow 拿到事件之后，会先将事件传递给图层树中距离最靠近 UIWindow 那一层最后一个 view，然后调用其 hitTest:withEvent: 方法。注意这里是先将视图传递给 view，再调用其 hitTest:withEvent: 方法，并遵循以下原则：

- 如果 point 不在这个视图内，则去遍历其他视图
- 如果 point 在这个视图内，但是这个视图还有子视图，那么将事件传递给子视图，并且调用子视图的 hitTest:withEvent:
- 如果 point 在这个视图内，并且这个视图没有子视图，那么 return self，即它就是那个最合适的视图
- 如果 point 在这个视图内，并且这个视图没有子视图，但是不想作为处理事件的 view，那么可以 return nil，事件由父视图处理

另外， UIView 有些情况下是不能接受触摸事件的：

- 不允许交互：userInteractionEnabled = NO
- 隐藏：如果把父控件隐藏，那么子控件也会隐藏，隐藏的控件不能接受事件
- 透明度：如果设置一个控件的 alpha < 0.01，会直接影响子控件的透明度。alpha 在 0 到 0.01 之间会被当成透明处理

注：如果父控件不能接受触摸事件，那么子控件就不可能接受到事件。

综上，我们可以得出 hitTest:withEvent: 方法的大致实现如下：

```
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    // 是否能响应 touch 事件
    if !isUserInteractionEnabled || isHidden || alpha <= 0.01 { return nil }
    if self.point(inside: point, with: event) {  // 点击是否在 view 内
        for subView in subviews.reversed() {
            // 转坐标
            let convertdPoint = subView.convert(point, from: self)
            // 递归调用，直到有返回值，否则返回 nil
            let hitTestView = subView.hitTest(convertdPoint, with: event)
            if hitTestView != nil {
                return hitTestView!
            }
        }
        return self
    }
    return nil
}
```

用一张图来表示 hitTest:withEvent: 的调用过程（图是 OC 语法）:

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzwaslm87j60ni0q876e02.jpg)

#### 4. 实战分析

关于`hitTest:`概念、实例和原理我们都分析了，现在我给大家出一个简单的题目：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzxctfakbj619v0m778c02.jpg)

我在 `PurpleDemoView`和`OrangeDemoView`这两个类中打印了 `hitTest`和`pointInside`的调用。

现在的问题是：分别点击 ①②③④ ，`PurpleDemoView`和`OrangeDemoView`是按什么顺序打印`hitTest`和`pointInside`的？

如果你只是搞清楚 `hitTest`和`pointInside`的概念，上面这个问题大概率是回答不上来的，还需要实操Debug。

##### 点击区域③

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzxi712mqj60ln03ut9902.jpg)

有没有发现，我们点击一次区域③，回调了两组 `hitTest`和`pointInside`，这是为什么呢？

为什么会回调两组呢？

我们在 `hitTest` 的地方进行断点，看看这两次调用的线程堆栈有什么不一样？

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzxoqif06j61fw0u0wnw02.jpg)

经过对比我们可以发现，0-22两者线程操作是完全一致的，而23处，第二次`hitTest`回调了`____updateTouchesWithDigitizerEventAndDetermineIfShouldSend_block_invoke`

含义是：更新触摸与数字化事件，并确定是否应该发送。

那么这两次相同`hitTest`的回调有什么区别呢？哪一次才是决定响应的最终view？

我们来做一个测试：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzyafp5x7j60o20o2q5x02.jpg)

当我们使用默认 `[super hitTest:point withEvent:event]`时，可以看到进行了 `onClick` 的回调。

接下来我们将在`hitTest`第一次回调中将`hit`改为`NO`，看看是否还会触发`onClick`回调：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzyccoaf6j60mu04k3z802.jpg)

我们发现，在第一次将回调`hit`改为`NO`，最终还是会对`onClick`进行响应。

接着我们在`hitTest`第二次回调中将`hit`改为`NO`，看看是否还会触发`onClick`回调：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzyf0g6abj60la05e0tk02.jpg)

我们惊奇地发现，并没有触发`OrangeDemoView onClick`回调，而是触发了 `OrangeDemoView.superView`的`onClick`回调，这表明在两次`hitTest`回调事件中，起决定性作用的是第二次的结果。

那第一次`hitTest`回调是什么作用呢？为什么要多此一举呢？

我们再回顾第一次回调`hitTest`的线程堆栈：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzyifh5lzj60u00xcjww02.jpg)

可以发现第一次主要是决策哪个Window进行事件响应，第二次`hitTest`的回调是决定 UIWindow 的哪个元素进行响应。

可能有小伙伴会觉得这步合成一步不可以吗？当然也是可以的，但这样设计我自己的理解是为了业务的隔离。

- Window 负责和 Application 层级进行响应对接，告诉Application这个事件是我Window的，我签收了，谢谢，你可以走了
- Window 内部的响应逻辑 和 Application 无关，只在内部决策，在内部下发事件时，可以做些加工的行为

##### 点击区域④

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzxyw7gldj60l5033jru02.jpg)

#### 点击区域①

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzxz6x4qyj60md04mmyc02.jpg)

#### hitTest 对 pointInside 的影响

如果我们在一个类中的 hitTest 明确返回的 view，那么就不会再触发该类的 pointInside 回调：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzy780sffj60p00ljac302.jpg)

### （二）Gesture Recognizer（手势识别）

确定了最合适的 view，接下来就是识别是何种事件，在触摸事件中，对应的就是何种手势。Gesture Recognizer（手势识别器）是系统封装的一些类，用来识别一系列常见的手势，例如点击、长按等。在上一步中确定了合适的 view 之后，UIWindow 会将 touches 事件先传递给 Gesture Recognizer(UIGestureRecognizer.h)，再传递给视图。可以自定义一个手势验证一下。

UIGestureRecognizer.h 头文件如下：

```

typedef NS_ENUM(NSInteger, UIGestureRecognizerState) {
    UIGestureRecognizerStatePossible,   // the recognizer has not yet recognized its gesture, but may be evaluating touch events. this is the default state
    
    UIGestureRecognizerStateBegan,      // the recognizer has received touches recognized as the gesture. the action method will be called at the next turn of the run loop
    UIGestureRecognizerStateChanged,    // the recognizer has received touches recognized as a change to the gesture. the action method will be called at the next turn of the run loop
    UIGestureRecognizerStateEnded,      // the recognizer has received touches recognized as the end of the gesture. the action method will be called at the next turn of the run loop and the recognizer will be reset to UIGestureRecognizerStatePossible
    UIGestureRecognizerStateCancelled,  // the recognizer has received touches resulting in the cancellation of the gesture. the action method will be called at the next turn of the run loop. the recognizer will be reset to UIGestureRecognizerStatePossible
    
    UIGestureRecognizerStateFailed,     // the recognizer has received a touch sequence that can not be recognized as the gesture. the action method will not be called and the recognizer will be reset to UIGestureRecognizerStatePossible
    
    // Discrete Gestures – gesture recognizers that recognize a discrete event but do not report changes (for example, a tap) do not transition through the Began and Changed states and can not fail or be cancelled
    UIGestureRecognizerStateRecognized = UIGestureRecognizerStateEnded // the recognizer has received touches recognized as the gesture. the action method will be called at the next turn of the run loop and the recognizer will be reset to UIGestureRecognizerStatePossible
};

UIKIT_EXTERN API_AVAILABLE(ios(3.2)) @interface UIGestureRecognizer : NSObject

// Valid action method signatures:
//     -(void)handleGesture;
//     -(void)handleGesture:(UIGestureRecognizer*)gestureRecognizer;
- (instancetype)initWithTarget:(nullable id)target action:(nullable SEL)action NS_DESIGNATED_INITIALIZER; // designated initializer

- (instancetype)init;
- (nullable instancetype)initWithCoder:(NSCoder *)coder;

- (void)addTarget:(id)target action:(SEL)action;    // add a target/action pair. you can call this multiple times to specify multiple target/actions
- (void)removeTarget:(nullable id)target action:(nullable SEL)action; // remove the specified target/action pair. passing nil for target matches all targets, and the same for actions

@property(nonatomic,readonly) UIGestureRecognizerState state;  // the current state of the gesture recognizer

@property(nullable,nonatomic,weak) id <UIGestureRecognizerDelegate> delegate; // the gesture recognizer's delegate

@property(nonatomic, getter=isEnabled) BOOL enabled;  // default is YES. disabled gesture recognizers will not receive touches. when changed to NO the gesture recognizer will be cancelled if it's currently recognizing a gesture

// a UIGestureRecognizer receives touches hit-tested to its view and any of that view's subviews
@property(nullable, nonatomic,readonly) UIView *view;           // the view the gesture is attached to. set by adding the recognizer to a UIView using the addGestureRecognizer: method

@property(nonatomic) BOOL cancelsTouchesInView;       // default is YES. causes touchesCancelled:withEvent: or pressesCancelled:withEvent: to be sent to the view for all touches or presses recognized as part of this gesture immediately before the action method is called.
@property(nonatomic) BOOL delaysTouchesBegan;         // default is NO.  causes all touch or press events to be delivered to the target view only after this gesture has failed recognition. set to YES to prevent views from processing any touches or presses that may be recognized as part of this gesture
@property(nonatomic) BOOL delaysTouchesEnded;         // default is YES. causes touchesEnded or pressesEnded events to be delivered to the target view only after this gesture has failed recognition. this ensures that a touch or press that is part of the gesture can be cancelled if the gesture is recognized

@property(nonatomic, copy) NSArray<NSNumber *> *allowedTouchTypes API_AVAILABLE(ios(9.0)); // Array of UITouchTypes as NSNumbers.
@property(nonatomic, copy) NSArray<NSNumber *> *allowedPressTypes API_AVAILABLE(ios(9.0)); // Array of UIPressTypes as NSNumbers.

// Indicates whether the gesture recognizer will consider touches of different touch types simultaneously.
// If NO, it receives all touches that match its allowedTouchTypes.
// If YES, once it receives a touch of a certain type, it will ignore new touches of other types, until it is reset to UIGestureRecognizerStatePossible.
@property (nonatomic) BOOL requiresExclusiveTouchType API_AVAILABLE(ios(9.2)); // defaults to YES

// create a relationship with another gesture recognizer that will prevent this gesture's actions from being called until otherGestureRecognizer transitions to UIGestureRecognizerStateFailed
// if otherGestureRecognizer transitions to UIGestureRecognizerStateRecognized or UIGestureRecognizerStateBegan then this recognizer will instead transition to UIGestureRecognizerStateFailed
// example usage: a single tap may require a double tap to fail
- (void)requireGestureRecognizerToFail:(UIGestureRecognizer *)otherGestureRecognizer;

// individual UIGestureRecognizer subclasses may provide subclass-specific location information. see individual subclasses for details
- (CGPoint)locationInView:(nullable UIView*)view;                                // a generic single-point location for the gesture. usually the centroid of the touches involved

@property(nonatomic, readonly) NSUInteger numberOfTouches;                                          // number of touches involved for which locations can be queried
- (CGPoint)locationOfTouch:(NSUInteger)touchIndex inView:(nullable UIView*)view; // the location of a particular touch

@property (nullable, nonatomic, copy) NSString *name API_AVAILABLE(ios(11.0), tvos(11.0)); // name for debugging to appear in logging

// Values from the last event processed.
// These values are not considered as requirements for the gesture.
@property (nonatomic, readonly) UIKeyModifierFlags modifierFlags API_AVAILABLE(ios(13.4)) API_UNAVAILABLE(tvos, watchos);
@property (nonatomic, readonly) UIEventButtonMask buttonMask API_AVAILABLE(ios(13.4)) API_UNAVAILABLE(tvos, watchos);

@end

// 分割线，这里指的是 UIGestureRecognizerDelegate ，代理
@protocol UIGestureRecognizerDelegate <NSObject>
@optional
// called when a gesture recognizer attempts to transition out of UIGestureRecognizerStatePossible. returning NO causes it to transition to UIGestureRecognizerStateFailed
- (BOOL)gestureRecognizerShouldBegin:(UIGestureRecognizer *)gestureRecognizer;

// called when the recognition of one of gestureRecognizer or otherGestureRecognizer would be blocked by the other
// return YES to allow both to recognize simultaneously. the default implementation returns NO (by default no two gestures can be recognized simultaneously)
//
// note: returning YES is guaranteed to allow simultaneous recognition. returning NO is not guaranteed to prevent simultaneous recognition, as the other gesture's delegate may return YES
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer;

// called once per attempt to recognize, so failure requirements can be determined lazily and may be set up between recognizers across view hierarchies
// return YES to set up a dynamic failure requirement between gestureRecognizer and otherGestureRecognizer
//
// note: returning YES is guaranteed to set up the failure requirement. returning NO does not guarantee that there will not be a failure requirement as the other gesture's counterpart delegate or subclass methods may return YES
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRequireFailureOfGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer API_AVAILABLE(ios(7.0));
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldBeRequiredToFailByGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer API_AVAILABLE(ios(7.0));

// called before touchesBegan:withEvent: is called on the gesture recognizer for a new touch. return NO to prevent the gesture recognizer from seeing this touch
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch;

// called before pressesBegan:withEvent: is called on the gesture recognizer for a new press. return NO to prevent the gesture recognizer from seeing this press
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceivePress:(UIPress *)press;

// called once before either -gestureRecognizer:shouldReceiveTouch: or -gestureRecognizer:shouldReceivePress:
// return NO to prevent the gesture recognizer from seeing this event
- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveEvent:(UIEvent *)event API_AVAILABLE(ios(13.4), tvos(13.4)) API_UNAVAILABLE(watchos);

@end

```

Gesture Recognizer 有一套自己的 touches 方法和状态转换机制。一个手势总是以 possible 状态开始，表明它已经准备好开始处理事件。从该状态开始，开始识别各种手势，直到它们到达 ended、cancelled 或 failed 状态。手势识别器会保持在其中的一个最终状态，直到当前事件序列结束，此时 UIKit 重置手势识别器并将其返回 possible 状态。

- 长按手势（UILongPressGestureRecognizer）
- 拖动手势（UIPanGestureRecognizer）
- 捏合手势（UIPinchGestureRecognizer）
- 响应屏幕边缘手势（UIScreenEdgePanGestureRecognizer）
- 轻扫手势（UISwipeGestureRecognizer）
- 旋转手势（UIRotationGestureRecognizer）
- 点击手势（UITapGestureRecognizer）

苹果将手势识别器分为两种大类型，一个是离散型手势识别器（Discrete Gesture Recognizer），一个是连续型手势识别器（Continuous Gesture Recognizer）。离散型手势一旦识别就无法取消，而且只会调用一次操作事件，而连续型手势会多次调用操作事件，并且可以取消。在以上手势中，只有点击手势（UITapGestureRecognizer）属于离散型手势。

##### 1. 离散型手势状态机

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzy3gkco8j60sa0kujsb02.jpg)

##### 2. 连续型手势状态机

连续型手势识别的状态转换一般可分为三个阶段：

- 1. 初始事件序列将手势识别器移动到 began 或 failed 状态
- 2. 后续事件将手势识别器移动到 changed 或 cancelled 状态
- 3. 最终事件将手势识别器移动到 ended 状态

![](https://tva1.sinaimg.cn/large/008i3skNgy1guzy41wkhcj60sz0oht9q02.jpg)

### （三）Response Chain（响应链，touch 事件传递）

#### 1. 什么是 UIResponder（响应者）

在 iOS 中，只有继承于 UIResponder 的对象、或者它本身才能成为响应者。很多常见的对象都可以相应事件，比如 UIApplication 、UIViewController、所有的 UIView（包括 UIWindow）。

UIResponder 提供了以下方法来处理事件：

```
typedef NSDictionary<NSAttributedStringKey, id> * _Nonnull(^UITextAttributesConversionHandler)(NSDictionary<NSAttributedStringKey, id> * _Nonnull);

typedef NS_ENUM(NSInteger, UIEditingInteractionConfiguration) {
    UIEditingInteractionConfigurationNone              = 0,
    UIEditingInteractionConfigurationDefault           = 1,      // Default
} API_AVAILABLE(ios(13.0));

@protocol UIResponderStandardEditActions <NSObject>
@optional
- (void)cut:(nullable id)sender API_AVAILABLE(ios(3.0));
- (void)copy:(nullable id)sender API_AVAILABLE(ios(3.0));
- (void)paste:(nullable id)sender API_AVAILABLE(ios(3.0));
- (void)select:(nullable id)sender API_AVAILABLE(ios(3.0));
- (void)selectAll:(nullable id)sender API_AVAILABLE(ios(3.0));
- (void)delete:(nullable id)sender API_AVAILABLE(ios(3.2));
- (void)makeTextWritingDirectionLeftToRight:(nullable id)sender API_AVAILABLE(ios(5.0));
- (void)makeTextWritingDirectionRightToLeft:(nullable id)sender API_AVAILABLE(ios(5.0));
- (void)toggleBoldface:(nullable id)sender API_AVAILABLE(ios(6.0));
- (void)toggleItalics:(nullable id)sender API_AVAILABLE(ios(6.0));
- (void)toggleUnderline:(nullable id)sender API_AVAILABLE(ios(6.0));

- (void)increaseSize:(nullable id)sender API_AVAILABLE(ios(7.0));
- (void)decreaseSize:(nullable id)sender API_AVAILABLE(ios(7.0));

- (void)updateTextAttributesWithConversionHandler:(NS_NOESCAPE UITextAttributesConversionHandler _Nonnull)conversionHandler API_AVAILABLE(ios(13.0));

@end

UIKIT_EXTERN API_AVAILABLE(ios(2.0)) @interface UIResponder : NSObject <UIResponderStandardEditActions>

@property(nonatomic, readonly, nullable) UIResponder *nextResponder;

@property(nonatomic, readonly) BOOL canBecomeFirstResponder;    // default is NO
- (BOOL)becomeFirstResponder;

@property(nonatomic, readonly) BOOL canResignFirstResponder;    // default is YES
- (BOOL)resignFirstResponder;

@property(nonatomic, readonly) BOOL isFirstResponder;

// Generally, all responders which do custom touch handling should override all four of these methods.
// Your responder will receive either touchesEnded:withEvent: or touchesCancelled:withEvent: for each
// touch it is handling (those touches it received in touchesBegan:withEvent:).
// *** You must handle cancelled touches to ensure correct behavior in your application.  Failure to
// do so is very likely to lead to incorrect behavior or crashes.
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
- (void)touchesEstimatedPropertiesUpdated:(NSSet<UITouch *> *)touches API_AVAILABLE(ios(9.1));

// Generally, all responders which do custom press handling should override all four of these methods.
// Your responder will receive either pressesEnded:withEvent or pressesCancelled:withEvent: for each
// press it is handling (those presses it received in pressesBegan:withEvent:).
// pressesChanged:withEvent: will be invoked for presses that provide an analog value
// (like thumbsticks or analog push buttons)
// *** You must handle cancelled presses to ensure correct behavior in your application.  Failure to
// do so is very likely to lead to incorrect behavior or crashes.
- (void)pressesBegan:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));
- (void)pressesChanged:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));
- (void)pressesEnded:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));
- (void)pressesCancelled:(NSSet<UIPress *> *)presses withEvent:(nullable UIPressesEvent *)event API_AVAILABLE(ios(9.0));

- (void)motionBegan:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
- (void)motionEnded:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));
- (void)motionCancelled:(UIEventSubtype)motion withEvent:(nullable UIEvent *)event API_AVAILABLE(ios(3.0));

- (void)remoteControlReceivedWithEvent:(nullable UIEvent *)event API_AVAILABLE(ios(4.0));

- (BOOL)canPerformAction:(SEL)action withSender:(nullable id)sender API_AVAILABLE(ios(3.0));
// Allows an action to be forwarded to another target. By default checks -canPerformAction:withSender: to either return self, or go up the responder chain.
- (nullable id)targetForAction:(SEL)action withSender:(nullable id)sender API_AVAILABLE(ios(7.0));

// Overrides for menu building and validation
- (void)buildMenuWithBuilder:(id<UIMenuBuilder>)builder API_AVAILABLE(ios(13.0));
- (void)validateCommand:(UICommand *)command API_AVAILABLE(ios(13.0));

@property(nullable, nonatomic,readonly) NSUndoManager *undoManager API_AVAILABLE(ios(3.0));

// Productivity editing interaction support for undo/redo/cut/copy/paste gestures
@property (nonatomic, readonly) UIEditingInteractionConfiguration editingInteractionConfiguration API_AVAILABLE(ios(13.0));

@end

@interface UIResponder (UIResponderKeyCommands)
@property (nullable,nonatomic,readonly) NSArray<UIKeyCommand *> *keyCommands API_AVAILABLE(ios(7.0)); // returns an array of UIKeyCommand objects<
@end

@class UIInputViewController;
@class UITextInputMode;
@class UITextInputAssistantItem;

@interface UIResponder (UIResponderInputViewAdditions)

// Called and presented when object becomes first responder.  Goes up the responder chain.
@property (nullable, nonatomic, readonly, strong) __kindof UIView *inputView API_AVAILABLE(ios(3.2));
@property (nullable, nonatomic, readonly, strong) __kindof UIView *inputAccessoryView API_AVAILABLE(ios(3.2));

/// This method is for clients that wish to put buttons on the Shortcuts Bar, shown on top of the keyboard.
/// You may modify the returned inputAssistantItem to add to or replace the existing items on the bar.
/// Modifications made to the returned UITextInputAssistantItem are reflected automatically.
/// This method should not be overridden. Goes up the responder chain.
@property (nonnull, nonatomic, readonly, strong) UITextInputAssistantItem *inputAssistantItem API_AVAILABLE(ios(9.0)) API_UNAVAILABLE(tvos) API_UNAVAILABLE(watchos);

// For viewController equivalents of -inputView and -inputAccessoryView
// Called and presented when object becomes first responder.  Goes up the responder chain.
@property (nullable, nonatomic, readonly, strong) UIInputViewController *inputViewController API_AVAILABLE(ios(8.0));
@property (nullable, nonatomic, readonly, strong) UIInputViewController *inputAccessoryViewController API_AVAILABLE(ios(8.0));

/* When queried, returns the current UITextInputMode, from which the keyboard language can be determined.
 * When overridden it should return a previously-queried UITextInputMode object, which will attempt to be
 * set inside that app, but not persistently affect the user's system-wide keyboard settings. */
@property (nullable, nonatomic, readonly, strong) UITextInputMode *textInputMode API_AVAILABLE(ios(7.0));
/* When the first responder changes and an identifier is queried, the system will establish a context to
 * track the textInputMode automatically. The system will save and restore the state of that context to
 * the user defaults via the app identifier. Use of -textInputMode above will supersede use of -textInputContextIdentifier. */
@property (nullable, nonatomic, readonly, strong) NSString *textInputContextIdentifier API_AVAILABLE(ios(7.0));
// This call is to remove stored app identifier state that is no longer needed.
+ (void)clearTextInputContextIdentifier:(NSString *)identifier API_AVAILABLE(ios(7.0));

// If called while object is first responder, reloads inputView, inputAccessoryView, and textInputMode.  Otherwise ignored.
- (void)reloadInputViews API_AVAILABLE(ios(3.2));

@end
```

可以看到`UIResponder`提供了我们平时最常用的`touchesBegan/touchesMoved/touchesEnded`方法。此外还有如下几个属性比较重要：

- isFirstResponder:判断该View是否为第一响应者。
- canBecomeFirstResponder:判断该View是否可以成为第一响应者。
- becomeFirstResponder:使该View成为第一响应者。
- resignFirstResponder:取消View的第一响应者。

如果我们将一个`view_A`先加在`view_B`上，然后又加到`view_C`上，那么`view_A.nextResponder`指的是`view_B`。

#### 2. 响应链

识别出手势之后，就要确定由谁来响应这个事件了，最有机会处理事件的对象就是通过 Hit-Test 找到的视图或者第一响应者，如果两个都不能处理，就需要传递给下一位响应者，然后依次传递，该过程与 Hit-Test 过程正好相反。Hit-Test 过程是从上向下（从父视图到子视图）遍历，touch 事件处理传递是从下向上（从子视图到父视图）传递。下一位响应者是由响应者链决定的，那我们先来看看什么是响应者链。
Response Chain，响应链，一般我们称之为响应者链。

在我们的 app 中，所有的视图都是按照一定的结构组织起来的，即树状层次结构，每个 view 都有自己的 superView，包括 controller 的 topmost view(即 controller 的 self.view)。当一个 view 被 add 到 superView 上的时候，它的 nextResponder 属性就会被指向它的 superView。当 controller 被初始化的时候，self.view(topmost view) 的 nextResponder 会被指向所在的 controller，而 controller 的 nextResponder 会被指向 self.view 的 superView，这样，整个 app 就通过 nextResponder 串成了一条链，这就是我们所说的响应者链。所以响应者链式一条虚拟的链，并没有一个对象来专门存储这样的一条链，而是通过 UIResponder 的属性串联起来的。

响应者链示意图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gv07paa5odj613l0lhq5102.jpg)

即（右图）：

- 初始视图（initial view）尝试处理事件，如果不能处理，则将事件传递给其父视图（superView1）
- superView1 尝试处理事件，如果不能处理，传递给它所属的视图控制器（viewController1）
- viewController1 尝试处理事件，如果不能处理，传递给 superView1 的父视图（superView2）
- superView2 尝试处理事件，如果不能处理，传递给 superView2 所属的视图控制器（viewController2）
- viewController2 尝试处理事件，如果不能处理，传递给 nil（新版本变更）
- Windows 的nextResponder是 Window Scene；
- Window Scene 的nextResponder是 Application
- UIApplication 尝试处理事件，如果不能处理，抛弃该事件

本文参考文章：

- [iOS概念攻坚之路（六）：事件传递与响应](https://juejin.cn/post/6844903865414844424)

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)