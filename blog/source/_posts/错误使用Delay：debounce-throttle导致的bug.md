---
title: 错误使用Delay：debounce/throttle导致的bug
date: 2021-10-26 22:11
tags: [debounce,throttle]
categories: [技术方案]
---

# 错误使用Delay：debounce/throttle导致的bug

上一节我们介绍了 debounce 和 throttle 的区别：[防抖（debounce）和节流（throttle）的区别](https://bninecoding.com/fang-dou-debounce-he-jie-liu-throttle-de-qu-bie.html)

今天我们更深入地聊一下。

## 一、throttle 的代码实现

```
- (BOOL)performSelector:(SEL)selector withObject:(id)object withMinInterval:(NSTimeInterval)minInterval {
    NSString *selectorStr = NSStringFromSelector(selector) ?: @"";
    BOOL shouldPerfrom;
    @synchronized(self) {
        NSMutableDictionary<NSString *, NSNumber *> *throttleDict = objc_getAssociatedObject(self, kGCThrottleDictKey);
        if (throttleDict == nil) {
            throttleDict = [NSMutableDictionary new];
            objc_setAssociatedObject(self, kGCThrottleDictKey, throttleDict, OBJC_ASSOCIATION_RETAIN);
        }
        NSTimeInterval lastTimeInterval = [[throttleDict objectForKey:selectorStr] doubleValue];
        NSTimeInterval nowInterval = [[NSDate date] timeIntervalSince1970];
        // [CUtility getCurrentClock]的时间有点长。。
        //避免用户更改时间 [[NSDate date] timeIntervalSince1970];
        shouldPerfrom = lastTimeInterval == 0 || (fabs(nowInterval - lastTimeInterval) > minInterval); // 用fabs，避免用户更改时间这种导致一直小于
        if (shouldPerfrom) {
            [throttleDict setObject:@(nowInterval) forKey:selectorStr];
        } else {
            MMInfo(@"%@ %@ ignored:%f - %f < %f", object, selectorStr, nowInterval, lastTimeInterval, minInterval);
        }
    }
    if (shouldPerfrom) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
        [self performSelector:selector withObject:object];
#pragma clang diagnostic pop
        return YES;
    } else {
        return NO;
    }
}
```

## 二、使用 debounce/throttle 带来的弊端

无论是 debounce 还是 throttle，我们都明白他们的主要应用是让调用不那么频繁。

然而这两种处理方式都是 丢掉调用方 的call事件。

我们回想下 debounce 做了什么？

debounce 的逻辑是：当有方法调用时，我就delay n秒再去执行，如果n秒内还有方法调用，那么就cancel掉上个延时，重新 delay n秒再去执行该方法。

但我们可以想象到，如果不断有对象调用该方法，会导致该方法一直 delay ，极端情况是该方法永远都无法执行，这就会导致bug。

然后我们再回想 throttle 做了什么？

throttle 的逻辑是：n秒内只执行1次该方法，如果n秒内有方法调用，那么就直接丢掉该方法。

那么问题来了，一般而言后调用该方法的对象，持有最新的状态，而throttle的逻辑会导致先执行旧的状态，丢掉新的状态。


## 三、debounce/throttle 如何选择

debounce 会执行最新的状态，但存在一直被delay的风险。

throttle 会执行相对较旧的状态，但存在丢掉最新状态的风险。

所以如果你的使用 delay（debounce/throttle）分散的方法是状态不相关的，比如只是 UI点击事件，那么建议使用 throttle。

如果你使用 delay（debounce/throttle）分散的方法是和状态相关的，这里有两种情况：

- 如果高频调用持续时间非常长，那么建议使用 throttle，但间隔要尽可能小，比如0.2
- 如果高频调用持续时间不长，瞬发时间段内调用比较频繁（比如界面刚打开，各种cgi回包刷新状态），那么可以使用 debounce


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)