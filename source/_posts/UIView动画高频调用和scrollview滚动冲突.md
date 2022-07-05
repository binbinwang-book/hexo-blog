---
title: UIView动画高频调用和scrollview滚动冲突
date: 2021-12-24 13:36
tags: [UIView,动画,scrollView]
categories: [iOS]
---

# UIView动画高频调用和scrollview滚动冲突

近期查了两个和动画相关的两个bug，都是比较隐秘的特性，清楚这两点绝对可以让你少踩很多坑。

## UIView动画高频调用

现在有如下一个函数：

```
- (void)runAnima {
    NSUInteger fromCount = self.count;
    self.count ++;
    NSUInteger toCount = self.count;
    [UIView animateWithDuration:3 animations:^{
        [self.scrollView setContentOffset:CGPointMake(20 * toCount, 0)];
    } completion:^(BOOL finished) {
        if (finished) {
            NSLog(@"fromCount:%lu toCount:%lu contentOffsetX:%f",(unsigned long)fromCount,toCount,self.scrollView.contentOffset.x);
        }
    }];
}
```

我们对这个函数for循环调用3次:

```
    for (int i = 0 ; i < 3; i ++) {
        [self runAnima];
    }
```

试得出Log打印出的日志内容，以及最终 scrollView contentOffset的值。

按照我们常规理解，打印的Log会按照 count ： 0 -> 1 -> 2 ，但实际并非如此：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxnus0eumxj314c0u00wi.jpg)

可以看到在动画还没结束时，如果重复调用该动画，那么会立即执行新加入动画的completion，但最终 scrollView contentOffset 是精确的。

所以这里需要注意的事情为：

如果你在使用 UIView animation 进行动画时，completion中有强依赖调用时序的变量，那么最好的方式是加一层保护：

```
- (void)runAnima {
    NSUInteger fromCount = self.count;
    self.count ++;
    NSUInteger toCount = self.count;
    [UIView animateWithDuration:3 animations:^{
        [self.scrollView setContentOffset:CGPointMake(20 * toCount, 0)];
    } completion:^(BOOL finished) {
        if (finished) {
            if (self.count == toCount) {
                NSLog(@"fromCount:%lu toCount:%lu contentOffsetX:%f",(unsigned long)fromCount,toCount,self.scrollView.contentOffset.x);
            }
        }
    }];
}
```

当然了，在 UIView animation 动画的 completion 中不使用变量，会是一个更好的选择。

## scrollView 滚动冲突

这个知识点的对应的bug是：

```
背景：scrollView被设置了 pageEnable，也就是允许按page为粒度进行滚动。

用户在滑动 scrollView 触发 pageEnable 系统滚动时，如果这时又有一个回调通过如下的方式主动修改了 scrollView 的 contentOffset：

[scrollView setContentOffset:xxx]

会发现系统的pageEnable滚动并没有打断，而且最终 scrollView 的contentOffset也并不是 xxx。
```

解决这个问题的方式很简单：

将`[scrollView setContentOffset:xxx]`修改为`[scrollView setContentOffset:xxx animated:NO]`即可，为什么呢？

参见 [Stack Overflow](https://stackoverflow.com/questions/3410777/how-can-i-programmatically-force-stop-scrolling-in-a-uiscrollview) 上的解释：

```
The generic answer is, that [scrollView setContentOffset:offset animated:NO] is not the same as [scrollView setContentOffset:offset] !

[scrollView setContentOffset:offset animated:NO] actually stops any running animation.
[scrollView setContentOffset:offset] doesn't stop any running animation.
Same for scrollView.contentOffset = offset: doesn't stop any running animation.
```

## 监听 scrollView 是否正在滚动

接上部分`scrollView 滚动冲突`处理完后，又发现了一个因为解决`scrollView 滚动冲突`导致的系统`scrollViewDidEndDecelerating:回调异常`的问题。

问题描述如下：

背景：scrollView 支持 pageEnable

用户对scrollView进行翻页滑动时，在scrollView系统滑动还没结束前，有其它的回调修改了 contentOffset ，这时候会立刻触发`scrollViewDidEndDecelerating:`回调，而实际上经过上面第2步的验证，我们知道 scrollView 本身是没有停止的。

也就是说`scrollViewDidEndDecelerating:`变得不可信了。

之前我们是通过 `scrollViewDidScroll:`和`scrollViewDidEndDecelerating:`维护一个标志位来判断`scrollView`是否在滚动的，那么既然`scrollViewDidEndDecelerating:`接口本身不可信，通过维护标志位判断`scrollView`是否在滚动也就变得不可行。

那么我们就会产生一个疑问：

怎么能判断一个scrollView是否正在滑动呢？

Google了一波，发现大家提供的方法要嘛在iOS高版本失效了，要嘛就不可依赖，最后自己查scrollView的api，发现了这三个property：

```
@property(nonatomic,readonly,getter=isTracking)     BOOL tracking;        // returns YES if user has touched. may not yet have started dragging
@property(nonatomic,readonly,getter=isDragging)     BOOL dragging;        // returns YES if user has started scrolling. this may require some time and or distance to move to initiate dragging
@property(nonatomic,readonly,getter=isDecelerating) BOOL decelerating;    // returns YES if user isn't dragging (touch up) but scroll view is still moving
```
遂构造了下面的方法来监测scrollView是否正在滑动，目前没发现有什么问题：

```
- (BOOL)timelineScrollViewIsScrolling {
    BOOL scrollViewIsBeTouched = self.timelineScrollView.isTracking || self.timelineScrollView.isDragging;
    BOOL scrollViewIsDecelerating = self.timelineScrollView.isDecelerating;
    if (scrollViewIsBeTouched || scrollViewIsDecelerating) {
        return YES;
    }
    return NO;
}
```

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)