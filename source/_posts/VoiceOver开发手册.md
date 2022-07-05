---
title: VoiceOver开发手册
date: 2021-09-14 10:01
tags: [VoiceOver,适老化]
categories: [iOS]
---

# VoiceOver开发手册

## 前言

近期处理VoiceOver适配问题时，在复杂界面处理遇到了一些棘手的问题，VoiceOver问题可参考的资料不多，所以我把自己踩过的坑记录如下。

本文将直接从开发的角度切入，不再介绍VoiveOver的实际意义。

本文描述操作路径的版本：

```
iPhone 12 Pro / iOS 14.7.1
iOS 14.5 Frameworks - UIKit
```

## 一、VoiceOver 操作指南

![](https://tva1.sinaimg.cn/large/008i3skNgy1gugbek5jpqj60hg0iwdje02.jpg)

再补充两个比较常用的手势：

- 切换App：四指滑动
- 拖动手势：单指双击并按住，拖动即可。

![四指滑动](https://tva1.sinaimg.cn/large/008i3skNgy1gughvncserg605k0b44qt02.gif)

手机开启了VoiceOver后，初次接触，会有几个比较高频遇到的问题：

### 1. 怎么再关闭 VoiceOver（如何给VoiceOver设置开关快捷键）

我们知道，打开VoiceOver的方式是：

`设置 - 辅助功能 - 旁白 - 打开`

但作为初次使用VoiceOver的问题是，使用手势切到了一个页面卡住了，不知道该如何返回，这时着急取消VoiceOver。

所以作为 VoiceOver 开发者，第一步你要做的是开启 「旁白 - 辅助功能快捷键」，步骤为：

`设置 - 辅助功能 - 辅助功能快捷键 - 旁白`

设置完毕后，你就可以连续按住手机侧边按钮3次实现 开/关 旁白VoiceOver 的功能。

### 2. 打开VoiceOver后，怎么切换到开发的Dev_App

#### 方法一（建议）：

配置好 「旁白 - 辅助功能快捷键」，打开 Dev_App ，连按3次侧边按钮，打开旁白

#### 方法二：

在设置中打开Voice，四指滑动屏幕，切换App。

### 3. 打开VoiceOver后，屏幕锁屏了，怎么在VoiceOver状态下解锁

#### 方法一：

如果你配置了「旁白 - 辅助功能快捷键」,那么连按3次侧边按钮，关闭VoiceOver，正常打开屏幕即可

#### 方法二：

如果你没有配置「旁白 - 辅助功能快捷键」,那么可以单指上滑后停止3秒钟，松手即可打开屏幕。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gughyynzlpg605k0b4b2e02.gif)

### 4. 打开VoiceOver后，不小心把控制面板拉下来了，怎么恢复控制面板？

和问题3解决方法一样。

## 二、开发指南

### （一） UIAccessibility API 解读

iOS中处理VoiceOver的关键类是`UIAccessibility.h`，相当于 VoiceOver 的 API，

抹去 `UIAccessibility.h` 中各项注释，我们得到如下内容：

```
@interface NSObject (UIAccessibility)

@property (nonatomic) BOOL isAccessibilityElement;

@property (nullable, nonatomic, copy) NSString *accessibilityLabel;

@property (nullable, nonatomic, copy) NSAttributedString *accessibilityAttributedLabel API_AVAILABLE(ios(11.0),tvos(11.0));

@property (nullable, nonatomic, copy) NSString *accessibilityHint;

@property (nullable, nonatomic, copy) NSAttributedString *accessibilityAttributedHint API_AVAILABLE(ios(11.0),tvos(11.0));

@property (nullable, nonatomic, copy) NSString *accessibilityValue;

@property (nullable, nonatomic, copy) NSAttributedString *accessibilityAttributedValue API_AVAILABLE(ios(11.0),tvos(11.0));

@property (nonatomic) UIAccessibilityTraits accessibilityTraits;

@property (nonatomic) CGRect accessibilityFrame;

UIKIT_EXTERN CGRect UIAccessibilityConvertFrameToScreenCoordinates(CGRect rect, UIView *view) API_AVAILABLE(ios(7.0));

@property (nullable, nonatomic, copy) UIBezierPath *accessibilityPath API_AVAILABLE(ios(7.0));

UIKIT_EXTERN UIBezierPath *UIAccessibilityConvertPathToScreenCoordinates(UIBezierPath *path, UIView *view) API_AVAILABLE(ios(7.0));

@property (nonatomic) CGPoint accessibilityActivationPoint API_AVAILABLE(ios(5.0));

@property (nullable, nonatomic, strong) NSString *accessibilityLanguage;

@property (nonatomic) BOOL accessibilityElementsHidden API_AVAILABLE(ios(5.0));

@property (nonatomic) BOOL accessibilityViewIsModal API_AVAILABLE(ios(5.0));

@property (nonatomic) BOOL shouldGroupAccessibilityChildren API_AVAILABLE(ios(6.0));

@property (nonatomic) UIAccessibilityNavigationStyle accessibilityNavigationStyle API_AVAILABLE(ios(8.0));

@property (nonatomic) BOOL accessibilityRespondsToUserInteraction API_AVAILABLE(ios(13.0),tvos(13.0));

@property (null_resettable, nonatomic, strong) NSArray<NSString *> *accessibilityUserInputLabels API_AVAILABLE(ios(13.0),tvos(13.0));

@property (null_resettable, nonatomic, copy) NSArray<NSAttributedString *> *accessibilityAttributedUserInputLabels API_AVAILABLE(ios(13.0),tvos(13.0));

@property(nullable, nonatomic, copy) NSArray *accessibilityHeaderElements UIKIT_AVAILABLE_TVOS_ONLY(9_0);

@property(nullable, nonatomic, strong) UIAccessibilityTextualContext accessibilityTextualContext API_AVAILABLE(ios(13.0), tvos(13.0));


@end

@interface NSObject (UIAccessibilityFocus)

- (void)accessibilityElementDidBecomeFocused API_AVAILABLE(ios(4.0));
- (void)accessibilityElementDidLoseFocus API_AVAILABLE(ios(4.0));

- (BOOL)accessibilityElementIsFocused API_AVAILABLE(ios(4.0));

- (nullable NSSet<UIAccessibilityAssistiveTechnologyIdentifier> *)accessibilityAssistiveTechnologyFocusedIdentifiers API_AVAILABLE(ios(9.0));

UIKIT_EXTERN __nullable id UIAccessibilityFocusedElement(UIAccessibilityAssistiveTechnologyIdentifier __nullable assistiveTechnologyIdentifier) API_AVAILABLE(ios(9.0));

@end

@interface NSObject (UIAccessibilityAction)

- (BOOL)accessibilityActivate API_AVAILABLE(ios(7.0));

- (void)accessibilityIncrement API_AVAILABLE(ios(4.0));
- (void)accessibilityDecrement API_AVAILABLE(ios(4.0));

typedef NS_ENUM(NSInteger, UIAccessibilityScrollDirection) {
    UIAccessibilityScrollDirectionRight = 1,
    UIAccessibilityScrollDirectionLeft,
    UIAccessibilityScrollDirectionUp,
    UIAccessibilityScrollDirectionDown,
    UIAccessibilityScrollDirectionNext API_AVAILABLE(ios(5.0)),
    UIAccessibilityScrollDirectionPrevious API_AVAILABLE(ios(5.0)),
};

- (BOOL)accessibilityScroll:(UIAccessibilityScrollDirection)direction API_AVAILABLE(ios(4.2));

- (BOOL)accessibilityPerformEscape API_AVAILABLE(ios(5.0));

- (BOOL)accessibilityPerformMagicTap API_AVAILABLE(ios(6.0));

@property (nullable, nonatomic, strong) NSArray <UIAccessibilityCustomAction *> *accessibilityCustomActions API_AVAILABLE(ios(8.0));
@end

@protocol UIAccessibilityReadingContent
@required

- (NSInteger)accessibilityLineNumberForPoint:(CGPoint)point API_AVAILABLE(ios(5.0));

- (nullable NSString *)accessibilityContentForLineNumber:(NSInteger)lineNumber API_AVAILABLE(ios(5.0));

- (CGRect)accessibilityFrameForLineNumber:(NSInteger)lineNumber API_AVAILABLE(ios(5.0));

- (nullable NSString *)accessibilityPageContent API_AVAILABLE(ios(5.0));

@optional
- (nullable NSAttributedString *)accessibilityAttributedContentForLineNumber:(NSInteger)lineNumber API_AVAILABLE(ios(11.0), tvos(11.0));
- (nullable NSAttributedString *)accessibilityAttributedPageContent API_AVAILABLE(ios(11.0), tvos(11.0));

@end

@interface NSObject(UIAccessibilityDragging)

@property (nullable, nonatomic, copy) NSArray<UIAccessibilityLocationDescriptor *> *accessibilityDragSourceDescriptors API_AVAILABLE(ios(11.0)) API_UNAVAILABLE(watchos, tvos);

@property (nullable, nonatomic, copy) NSArray<UIAccessibilityLocationDescriptor *> *accessibilityDropPointDescriptors API_AVAILABLE(ios(11.0)) API_UNAVAILABLE(watchos, tvos);


@end
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1guggivfcf9j60fa0e8mxi02.jpg)

通过阅读`UIAccessibility.h`我们可以发现这个类针对 `NSObject`新增了4个Category，声明了一个 Notification：

- UIAccessibility ：VoiceOver基本设置的属性
- UIAccessibilityFocus ：VoiceOver聚焦相关逻辑
- UIAccessibilityAction ：处理和 VoiceOver 的手势冲突问题
- UIAccessibilityPostNotification ： 抛VoiceOver的通知，控制VoiceOver行为

#### 1. UIAccessibility

UIAccessibility 这个 Category 的官方解释为：

```
 UIAccessibility is implemented on all standard UIKit views and controls so
 that assistive applications can present them to users with disabilities.
 
 Custom items in a user interface should override aspects of UIAccessibility
 to supply details where the default value is incomplete.
 
 For example, a UIImageView subclass may need to override accessibilityLabel,
 but it does not need to override accessibilityFrame.
 
 A completely custom subclass of UIView might need to override all of the
 UIAccessibility methods except accessibilityFrame.
```

重点关注最后一句话：一个完全自定义的view可能需要覆写UIAccessibility这个Category中除了accessibilityFrame之外的所有方法。

可见 UIAccessibility 这个 Category 在 VoiceOver 中的重要性。

##### （1） isAccessibilityElement （property） ⭐️⭐️⭐️

作为 VoiceOver 最关键的一个property，只有 isAccessibilityElement 才会响应 VoiceOver 选中事件。

关于 isAccessibilityElement 还有两个非常重要的特性：

- 默认值：UIView 中，只有 UILabel、UIButton 的 isAccessibilityElement 默认是 YES，其余 UIView.isAccessibilityElement 默认是 NO。
- 消息传递：view.superView.isAccessibilityElement 为 YES ，无论 view.isAccessibilityElement 怎么设置，VoiceOver 都将不会响应到 view，而只会响应 view.superView

Tips：关于 VoiceOver 的消息传递，我们在讲完API后会深入探讨。

##### （2） accessibilityLabel （property） ⭐️⭐️⭐️

当我们选中 View ，VoiceOver 读出的内容就是 accessibilityLabel 的内容。

##### （3） accessibilityHint （property）⭐️⭐️

一个简短的本地化短语，用于描述作用于元素上的操作所得到的结果。 例如“添加标题”或“查看购物车”

##### （4）accessibilityValue（property）⭐️⭐️

某UI元素的当前值，并且这个值无法由label表示出来。例如，某个slider的accessibilityLabel可能是“速度”，但其当前值可能为“50％”

##### （5）accessibilityTraits（property）⭐️⭐️

一个或多个单独特征的组合，每个特征描述元素的某一个方面，包括状态、行为或用法。这些都定义成了UIAccessibilityTrait的某一个常量。

##### （6） accessibilityFrame （property）⭐️

屏幕中某元素的坐标和大小。这个属性比较重要，但实际开发过程中我们很少会显示的设置accessibilityFrame。

##### （7） accessibilityPath （property）

手动画贝塞尔曲线决定响应的范围，和 accessibilityFrame 一样，基本用不到，但要知道有这个功能。

##### （8）accessibilityActivationPoint （property）

当你点击元素时，返回的point，默认值是 accessibilityFrame 的 mid-point。

##### （9）accessibilityLanguage （property）

VoiceOver 读内容时，使用的语言。常规情况下我们不需要显示地设置accessibilityLanguage，它会根据系统语言读出来。

##### （10） accessibilityElementsHidden （property）

设置一个view能否响应 VoiceOver ，isAccessibilityElement 是针对当前 View 的，而 accessibilityElementsHidden 是针对当前 View 和其所有 subView 的。

如果我们需要把一个ViewController的根View以及内部所有子View都不支持VoiceOver,可以这么写:

`self.view.accessibilityElementsHidden = YES;`

VoiceOver 本身的设置体系就比较散，不建议使用 accessibilityElementsHidden ，不然会导致 superView 的设置导致所有的子view都受到影响，可能会影响到其它同学的业务。

慎用。

##### （11） accessibilityViewIsModal （property）⭐️⭐️

试想这样一个场景：A present 出半屏B，我们预期是 VoiceOver 只在 半屏B 响应，只有关闭了B，VoiceOver 才可以去感知 A。

这时如果把 半屏B.accessibilityViewIsModal 设置为 YES，就可以实现上面的效果。

##### （12） shouldGroupAccessibilityChildren （property）⭐️

可以通过 shouldGroupAccessibilityChildren 将一系列子view聚合在一起，调整 VoiceOver 响应的顺序。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gugfhwdo0xj60l60aswei02.jpg)

比如上面这个view，如果我们不做任何调整，VoiceOver 读取的顺序是：

`Label 1 -> Label 2 -> Button -> Label 3`

如果我们把 label 1、2、3 放到一个父view labelContainer 中，并设置 labelContainer.shouldGroupAccessibilityChildren 为 YES，

那么 VoiceOver 的顺序将调整为：

`Label 1 -> Label 2 -> Label 3 -> Button`

还没有罗列的属性如下，当你觉得你遇到的问题实在是无法用现有API解决，可以看看下面这些比较偏僻的API：

```
accessibilityNavigationStyle
accessibilityRespondsToUserInteraction
accessibilityUserInputLabels
accessibilityAttributedUserInputLabels
accessibilityHeaderElements
accessibilityTextualContext
```

⭐️⭐️⭐️问题：Traits、label、Value、Hint 都是 NSString 类型，那么如果都设置上，会都读出来吗？读的顺序如何呢？

按下面这个流程设置：

```
    self.likeBtn.accessibilityLabel = @"点赞";
    self.likeBtn.accessibilityTraits = UIAccessibilityTraitSelected;
    self.likeBtn.accessibilityValue = @"99";
    self.likeBtn.accessibilityHint= @"给内容点赞";
```

VoiceOver 读出的顺序是： 已选定 - 点赞 - 99 - 给内容点赞，

也即： `Traits - label - value - hint`

所以如果你只是给view设置一个读音，使用上面任何一个元素都是可以实现的，但要注意的是：

开发应面向未来，应该把读音放到合适的字段，不要错误占用了字段。

#### 2. UIAccessibilityFocus

此 Category 主要用于处理 VoiceOver 下的聚焦问题。

##### （1） accessibilityElementDidBecomeFocused （method）

成为焦点后的回调。

##### （2）accessibilityElementDidLoseFocus （method）

失去焦点后的回调。

##### （3）accessibilityElementIsFocused （method）

获取当前是不是VoiceOver的焦点。

#### 3. UIAccessibilityFocus

##### （1） accessibilityActivate （method）

单指轻点两次的回调，默认的双击之后会触发的函数。一般可以通过覆写这个方法来解决手势冲突问题。

##### （2）accessibilityScroll: （method）

处理和控制 VoiceOver 滚动方向。

##### （3） accessibilityPerformEscape （method）

试想这样一个场景：A present 出模态B 视图，当A消失时，如果实现模态B的同时消失？

只需要在模态B中将`accessibilityPerformEscape`设置为YES即可。

#### 4. UIAccessibilityPostNotification

UIAccessibilityPostNotification(UIAccessibilityNotifications, view);

```
UIAccessibilityNotifications

UIAccessibilityScreenChangedNotification
UIAccessibilityLayoutChangedNotification
UIAccessibilityAnnouncementNotification
UIAccessibilityPageScrolledNotification
UIAccessibilityPauseAssistiveTechnologyNotification
UIAccessibilityResumeAssistiveTechnologyNotification

```

##### （1）UIAccessibilityScreenChangedNotification

新元素出现调用，或在 appear 生命周期处调用。

使用 `UIAccessibilityPostNotification(UIAccessibilityScreenChangedNotification, view)` 可以将 VoiceOver 的光标移动到 view

##### （2）UIAccessibilityLayoutChangedNotification

触发layout调用。

使用 `UIAccessibilityPostNotification(UIAccessibilityLayoutChangedNotification, view)` 可以将 VoiceOver 的光标移动到 view

##### （3）UIAccessibilityAnnouncementNotification

使用 `UIAccessibilityPostNotification(UIAccessibilityAnnouncementNotification, @"读出的内容")`，可以读出相应的内容，实现自定义时间点来读出相应内容。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)