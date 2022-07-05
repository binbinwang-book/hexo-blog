---
title: present半透明渐出的VC（含知识延伸）
date: 2021-08-07 18:38
tags: [present,modalStyle,presentation,transition]
categories: [iOS]
---

# present半透明渐出的VC（含知识延伸）

## 前言

我们普通的present效果是：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8el3h70hg30ay0muahe.gif)

有时候我们present的是想做成半透明，而且是渐出（dissolve）的效果的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8elrqgz5g30ay0mudqq.gif)

这个功能其实挺简单，实现起来就3步：

- 设置 presentation 的 modalStyle
- 设置 transition 的 modalStyle
- 设置背景色（半透明）

```
- (instancetype)init
{
    self = [super init];
    if (self) {
        self.modalPresentationStyle = UIModalPresentationCustom;
        self.modalTransitionStyle = UIModalTransitionStyleCrossDissolve;
        self.view.backgroundColor = [UIColor colorWithRed:0.3 green:0.3 blue:0.5 alpha:0.8];
    }
    return self;
}
```

基于这个功能，我想扩展聊一下这里的各项配置的含义。

## 一、modalPresentationStyle

关于`modalPresentationStyle`，官方文档的说明是：


Defines the presentation style that will be used for this view controller when it is presented modally. Set this property on the view controller to be presented, not the presenter.
 
If this property has been set to UIModalPresentationAutomatic, reading it will always return a concrete presentation style. By default UIViewController resolves UIModalPresentationAutomatic to UIModalPresentationPageSheet, but system-provided subclasses may resolve UIModalPresentationAutomatic to other concrete presentation styles. Participation in the resolution of UIModalPresentationAutomatic is reserved for system-provided view controllers.

Defaults to UIModalPresentationAutomatic on iOS starting in iOS 13.0, and UIModalPresentationFullScreen on previous versions. Defaults to UIModalPresentationFullScreen on all other platforms.

这段话需要提炼的重点是 iOS13 之后，`modalStyle`默认值是`Automatic`，而`Automatic`在iOS13默认会转成`PageSheet`的style，之前的版本都是默认`UIModalPresentationFullScreen`。


对应的枚举值是：

```
typedef NS_ENUM(NSInteger, UIModalPresentationStyle) {
    UIModalPresentationFullScreen = 0,
    UIModalPresentationPageSheet API_AVAILABLE(ios(3.2)) API_UNAVAILABLE(tvos),
    UIModalPresentationFormSheet API_AVAILABLE(ios(3.2)) API_UNAVAILABLE(tvos),
    UIModalPresentationCurrentContext API_AVAILABLE(ios(3.2)),
    UIModalPresentationCustom API_AVAILABLE(ios(7.0)),
    UIModalPresentationOverFullScreen API_AVAILABLE(ios(8.0)),
    UIModalPresentationOverCurrentContext API_AVAILABLE(ios(8.0)),
    UIModalPresentationPopover API_AVAILABLE(ios(8.0)) API_UNAVAILABLE(tvos),
    UIModalPresentationBlurOverFullScreen API_AVAILABLE(tvos(11.0)) API_UNAVAILABLE(ios) API_UNAVAILABLE(watchos),
    UIModalPresentationNone API_AVAILABLE(ios(7.0)) = -1,
    UIModalPresentationAutomatic API_AVAILABLE(ios(13.0)) = -2,
};
```

### （一）fullScreen

在各种 Size Class 情况下都是全屏展示

执行 present 操作的控制器的view和它的subViews，在 present 完成后都会被从当前视图层级`移除`（注意，移除是重点!）

### （二）pageSheet

被推出视图部分的遮盖下层视图

其宽度总是为该设备竖屏时候的宽度（不可变），高度则为当前设备方向的屏幕高度（可变，其实还要去掉状态栏的高度）

### （三）formSheet

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8dwnp8whj30zs0m83zg.jpg)

被推出视图大小比屏幕的小，且总是居中显示

在横屏时，如果弹出了键盘，视图位置会跟着上移

可以设置被推出视图的preferredContentSize来设置它的大小

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8czxfwytj30lk0tdwey.jpg)

这里设置了preferredContentSize = CGSize(width: 200, height: 200)。

Tips：formSheet 的格式也挺常见，我写了个demo，代码和效果如下：

```
    BNPresentDissolveViewController* modalController = [[BNPresentDissolveViewController alloc]init];
#     modalController.modalPresentationStyle =  UIModalPresentationFormSheet;
    modalController.preferredContentSize = CGSizeMake(200, 200);
    [self presentViewController:modalController animated:YES completion:nil];
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8dzg0a4tj31410u0wf4.jpg)

需要注意的是！！！ FormSheet 只有在iPad才生效，如果是 iPhone 想实现 FormSheet 的效果，得自己用 fullScreen 写。

所以整个项目工程里基本没有人使用 UIModalPresentationFormSheet ，但苹果API里又没说 UIModalPresentationFormSheet 只能给 iPad 用，而实际上 UIModalPresentationFormSheet 就是只在iPad上生效，这有点操蛋。

### （四）currentContext

- 可以用在 iPadUISplitViewController中，指定单独覆盖屏幕单侧的控制器；popover方式展示的控制器，再用该方式 present 出下一视图
- 在执行 present 操作的控制器的控制器层级中往上查找，如果某个控制器的definesPresentationContext == true则它来 present，假如没有一个为true，那么则由 window.rootController来 present
- 执行 present 操作的控制器的view和它的subViews，在 present 完成后都会被从当前视图层级移除

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8d2xw4wlj30lk0tdglr.jpg)

### （五）custom

自定义模式，需要实现UIViewControllerTransitioningDelegate的相关方法，并将presented VC的transitioningDelegate 设置为实现了UIViewControllerTransitioningDelegate协议的对象。

如果present时需要做自定义动画的话，modalStyle可以选择 custom。且选择 custom 这种 style， A present B 时，也不会把 A 销毁掉。

### （六）overFullScreen

基本和fullScreen一致。只是 present 完成后，不会移除执行 present 操作的控制器的view和它的subViews。如果presentedViewController.view是有透明度的，底层视图就可以得以显示。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8d15asa1j30lk0td74u.jpg)

### （七）overCurrentContext

基本和currentContext一致。只是 present 完成后，不会移除执行 present 操作的控制器的view和它的subViews。如果presentedViewController.view是有透明度的，底层视图就可以得以显示。

### （八）popover

可以天然实现一种缩放的交互，如果 A present B ，可以让A产生一种缩放的效果，非常不错。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8en3q6okg30ay0mukjl.gif)


## 二、modalTransitionStyle

关于 transition 的 modalStyle ，官方定义是：

Defines the transition style that will be used for this view controller when it is presented modally. Set this property on the view controller to be presented, not the presenter.  
  
Defaults to UIModalTransitionStyleCoverVertical.

modalTransitionStyle 对应的枚举是：

```
    UIModalTransitionStyleCoverVertical:在垂直方向上，显示的时候从下往上；关闭的时候从上往下

    UIModalTransitionStyleFlipHorizontal:水平翻转，显示的时候从右向左；关闭的时候从左往右

    UIModalTransitionStyleCrossDissolve:淡入淡出

    UIModalTransitionStylePartialCurl:边角翻页效果
```

我们来看一下不同 style 的 transition 效果吧

### （一）UIModalTransitionStyleCoverVertical

就是很普通的present，从下向上。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8el3h70hg30ay0muahe.gif)

### （二）UIModalTransitionStyleFlipHorizontal

有点花里胡哨..不过也挺好玩的。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8eo8kon1g30ay0muwtg.gif)

### （三）UIModalTransitionStyleCrossDissolve

我们预期的渐变功能：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8elrqgz5g30ay0mudqq.gif)

### （四）UIModalTransitionStylePartialCurl

这个 transitionStyle 必须匹配 FullScreen 来使用，不然会crash：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8eh6wshaj316a0dzdi5.jpg)

看起来就是翻书的动画，也挺有趣：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8epnu0qqg30ay0mu4j9.gif)

Tips：本来还期待着系统可以提供 从右向左 present 的 transition ModalStyle，结果没有，看来下次遇到还是得自己写（当然也不麻烦）。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)