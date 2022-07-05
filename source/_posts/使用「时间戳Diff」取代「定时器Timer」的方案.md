---
title: 使用「事件驱动&时间戳Diff」取代「定时器Timer」的方案
date: 2021-07-12 18:29
categories: 技术方案
tags: [技术方案]
---

# 使用「事件驱动&时间戳Diff」取代「定时器Timer」的方案

## 看完文章你会收获什么？

1. 什么时候适合用 Timer ，什么时候不适合用 Timer

2. 如何用「时间戳Diff方案」取代「定时器Timer」方案？

理解难度： 3/5 ⭐️⭐️⭐️

工程实用性： 4/5 ⭐️⭐️⭐️⭐️

## 引言

近期接到了一个产品的需求，脱敏后需求描述如下：

```
用户进到 A 场景后，开启计时，每隔100分钟展示弹框提醒用户去喝水；

用户退出 A 场景 / App 退后台 / 杀 App，计时要暂停；

用户重新进入 A 场景 / App 恢复前台且处在A场景，计时要恢复。
```

## 纯Timer实现技术方案

这个需求刚一接手看起来是没有什么难度的，按照 需求 映射成技术方案，然后再考虑需求文档里没有说明的技术细节，可以设计如下的技术方案：

>> 0. 开启一个Timer，进行定时轮询
>> 1. 特定场景对 Timer 进行 暂停/恢复 的逻辑
>> 2. 要处理好 Timer 时间间隔落本地的逻辑

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsebpqomy6j30fg0gyq3j.jpg)

这个方案看起来实现了产品的需求，但在体验过程中我发现了一个问题：

定时提醒弹框展示地过于突兀，用户可能正在专注看一个视频，100分钟倒计时结束突然弹框，影响用户体验。

所以我和产品argue了一下方案，将「100分钟倒计时结束就弹框」改为「用户通过手势触发倒计时校验，如果倒计时结束，就弹框」。

## 事件驱动Timer校验的方案

这个方案的好处是不会在用户浏览过程中突然弹框，而是在用户切换界面/点击暂停/点赞等事件时，再去检查 Timer 倒计时是否结束，如果结束则触发 Drink弹框 提醒，流程图描述如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsebwahb5sj30hx0fm0tn.jpg)

这个方案实现后，不会再突兀出现弹框的问题，但作为一名开发，用户在进入某个场景后，后台始终运行一个 timer ，总觉得是对资源的一种浪费，这时我进一步思考，有没有不通过 Timer 也可以实现倒计时的功能？

思考后，发现 「事件驱动 & 时间戳Diff」 可以非常完美地替换 「定时器Timer」的方案，方案描述如下。

## 「事件驱动 & 时间戳Diff」替换「定时器Timer」

既然我们已经使用「事件驱动」去做超时逻辑地校验，那么每两个事件之间的时间间隔我们是可以通过时间戳作差计算出来的，只要我们在每次触发事件检查时，累积每个事件检查之间的间隔，就可以完整地计算出总浏览时长，流程描述图如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gsec34z4xtj30j903xjrk.jpg)

关键代码如下：

```
- (void)updatePassSeconds {
    if (self.lastTimeStamp <= 0) {
        return;
    }

    if ([self _notInTimerScene]) {
        return;
    }
    self.passSeconds += ([CUtility genCurrentTimeInMsFrom1970] - self.lastTimeStamp) / 1000;
    self.lastTimeStamp = [CUtility genCurrentTimeInMsFrom1970];
}

- (void)triggerDrinkCheckLogic {
    if (![self _notInTimerScene]) {
        return;
    }

    [self updatePassSeconds];

    BOOL shouldTriggerDrinkAction = self.passSeconds > 100;
    if (shouldTriggerDrinkAction) {
        [self _triggerDrinkAction];
    }
}

```

只需要在每一个事件行为（比如 hitTest：）调用一下 `triggerDrinkCheckLogic ` 方法，就可以完成浏览总时长的累计计算。

至此，终于得出一个 产品满意 & 用户满意 & 技术满意 的方案。

## 总结

工作两年也完成了N次大规模使用`Timer`的需求方案，目前来看使用「事件驱动&时间戳Diff」都是更佳的选择，这促使着我思考究竟`Timer`还有没有可以应用的场景？

答案是有的。

如果你需要 n秒 间隔的回调，这种方式使用 Timer 是最佳的。

而如果你只是需要校验时间间隔，那么「事件驱动 & 时间戳Diff」相比「定时器Timer」更加优秀。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)