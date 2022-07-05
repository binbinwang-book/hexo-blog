---
title: 导航栏做动画时view展示不全
date: 2021-08-03 01:33
tags: [导航栏bug]
categories: [技术方案]
---

# 导航栏做动画时view展示不全

## 1.基本样式

自2020年5月视频号切换到3tab样式下，导航栏样式也从简单的单title样式变成了多tab样式，上周在处理导航栏红点时又遇到一个被裁断的问题，遂结合此篇小伎俩把我在处理视频号导航栏上遇到的问题，一一描述。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ykhz224j30ne0ec3yk.jpg)


构建3tab的UIView，每个tab平分titleView，接着将导航栏的titleView进行替换，导航栏默认居中展示，搞定！

第一个版本做demo时确实是按上面的流程处理的，但过完Demo正式上线前，遇到了诸多适配的问题。

## 2.小屏机 || 多语言 适配方案

问题一

测试同学报来的第一个bug是小屏机的问题，我们针对titleView布局时采用的是默认居中的方案，但实际上导航栏两边的UI元素并不对称：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2yko4fjbj30n80du3yn.jpg)


所以在小屏机上会出现 「推荐」 已经和 「🔍」重叠了，但实际上 「关注」 距离左边的 「<」 还有一定的距离，此为其一。

问题二

测试同学报来的第二个bug是多语言下的bug，尽管在中文下「关注」「朋友❤」「推荐」是左右两边tab的文字长度是一致的，但在多语言下每个tab的翻译长短不一。

只要我们采用了平分titleView或者按鼓励比例切割titleView的方式，就存在在某一种语言下某个tab翻译过长，导致 翻译过长的tab 英文字体被缩减，和其它tab下的文字大小不一。


针对以上出现的情况，我设计了如下的适配方案，伪代码描述如下


```
按照正常字体大小计算三个按钮的宽度

if(三个按钮宽度超过self.view总宽度) {
	按相同的比例缩放字体大小，缩放字体后再尝试下居中
	if(居中后会超出 左/右 边界(指leftBarItem/rightBarItem)) {
		放弃居中，采用 关注button 左对齐的方式进行布局
	}else{
		完成布局
	}
}else{
	if(是小屏机){
		采用「推荐button」右对齐的方式
	}else{
		进行居中操作
			if(居中后会超出 左/右 边界(指leftBarItem/rightBarItem)) {
				放弃居中，采用 关注button 左对齐的方式进行布局
			}else{
				完成布局
			}
	}
}
```

通过这样一套方案，解决了导航栏元素过多与小屏机宽度小、多语言翻译长的矛盾。


## 3.滑动屏幕切tab背景下导航栏的动画处理
在T5版本全屏浏览逐渐放开的阶段，交互提出了左右滑动屏幕来切tab的需求，那很好处理，直接 pagingEnabled/scrollEnabled = YES 即可。

但滑动屏幕切tab的功能做出来后，导航栏的动画切换却出了问题：

在还只能点击导航栏切换tab的时候，我是在点击事件松手那一刻切换导航栏tab的选中态和非选中态，一瞬间的操作，用户不觉得有什么问题。

但在用户滑动屏幕切tab时，就会看到一个非常明显的 A tab -> B tab 切换的一个动画，也即滑动屏幕松手时：
- 下划线会突然从A tab 滑动到 B tab，这和用户缓慢切换屏幕的手势没有对应起来
- A tab的字体会突然变小，A tab的颜色会突然变浅
- B tab的字体会突然变大，B tab的颜色会突然加深

这种突变的效果不仅交互看不过去，我也看不过去，接着我通过类比抖音和快手滑动屏幕切tab的交互，发现了如下规律：
在滑动屏幕时，有如下几个UI要跟着一起变更：
- tab下的下划线位置，长短
- fromTab的字体和颜色
- toTab的字体和颜色

既然总结出规律，那就好处理了，在滑动屏幕时，将屏幕滚动的contentOffset传到我们自定义的导航栏titleView中，
导航栏titleView的UI元素根据contentOffset来进行相应的变更，如此设计变完成了滑动屏幕切tab时导航栏渐变的动画。


## 4.红点兼容方案处理

### 4.1 导航栏基本概念补充

这里在阐述红点兼容方案前，我们大致回顾一下导航栏的架构。

UINavigationController的view视图结构:

```
-- UIView: frame = (0 0; 414 896);
    -- UINavigationTransitionView: frame = (0 0; 414 896);
    -- UINavigationBar: frame = (0 44; 414 44);
```

UINavigationTransitionView 是 UINavigationController 转场动画的容器视图，也是红点兼容处理故事的主角。

UINavigationController 的所有子视图控制器 共用一个UINavigationbar，UINavigationbar 的转场效果由参与转场的两个视图控制器对导航栏的设置决定。

导航栏的定制化可分为导航栏渲染配置、以及导航栏控件配置两个部分。

但涉及到具体的导航栏架构，在iOS11时方案出现了变化：

iOS 11以前：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2yl0641bj30s40bhq4n.jpg)


iOS 11之后：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2yl6w2obj312g0boacf.jpg)

UINavigationItemView -> _UINavigationBarContentView
内容视图的变化是导致之前设置样式失效的最大原因，iOS11所有的UIBarButtonItem都是加载到新的_UIButtonBarStackView上的，而_UIButtonBarStackView默认在5.5英寸机型有20px其余机型为16px的边距，UIBarButtonSystemItemFixedSpace也无法使用了。


### 4.2 导航栏新增红点遇到的问题和解决方案

上周我收到一个需求：需要把视频号消息的红点透传到发现页，评估后发现我需要在个人中心的小人头右上角加一个 数字红点。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ylemytcj30u10iz419.jpg)

加完之后mock完红点的路径，发现一切完美，但在我处于视频号3tab主页正要策划返回发现页时，发现了一个大问题：红点被裁断了

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ym4gsggj30no0cgdgh.jpg)

我突然转念一想，美团外卖也有类似的红点位置，我看看他们会不会有问题，于是我打开美团一看，完了，他们也有bug：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ym9wjzjj30o40b63z6.jpg)

----

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ymsclynj30o40b63z6.jpg)


别人没解决掉的，那就只能我们自己来解决了，下面是我分析的流程

#### 第一阶段：

「数字红点」是加到「个人中心」 MMBarButtonItem 上的，bug刚一被我发现时，我层一度怀疑是 MMBarButtonItem 自身的属性裁掉了它的subView，

于是我写了如下的demo代码验证：

MMBarButtonItem.layer.mastToBounds = NO;
MMBarButtonItem.layer.zPosition = 100;

但是问题还是存在，再思考🤔

#### 第二阶段：

会不会和导航栏左右留的16px间距有关系呢？因为看到被裁的部分也是16px，接着我分析正常状态和异常状态的视图结构：

正常状态：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ymye8cgj30u10iz419.jpg)


异常状态：
![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2yn64yosj311q0ov78i.jpg)



我们可以看到，异常状态下 MMBarButtonItem 已经不存在了，这时我突然意识到这种场景下已经是 UINavigationTransitionView 通过制造的 snapshot ，在执行动画了，也即：UINavigationTransitionView 在制造截图的 snapshot 时，裁到 MMBarButtonItem stackView时，是没有保留右边16px的。

这时我想到在侧滑返回时，会触发 viewWillDisappear ，那我们在 viewWillDisAppear 时，隐藏掉 「数字红点」 可行吗？

经过我的尝试，发现即使我在 viewWillDisappear 隐藏了「数字红点」，还是会出现被裁的问题。这说明了什么？说明了两件事：

- 侧滑返回时，导航栏的样式是假的，是snapShot，不是我们加上去的view
- UINavigationTransitionView 生成 snapShot 的时间要先于 viewWillDisappear


#### 第三阶段：

既然受限于 MMBarButtonItem stackView 左右16px的约束，那我如果把 「数字红点」 直接加到 navigationbar 上，应该就没问题了吧？

帖一下我们上面发的 navigationController 的架构：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2ynewzh4j314e0ncq5t.jpg)

可以看到 navigationbar 和 UINavigationTransitionView 是同级关系，所以如果我把「数字红点」加到了 navigationbar 上，

UINavigationTransitionView 在生成 snapShot 时，是不会把 navigationbar 上自定义UI控件包含进去的。

通过代码验证发现，将「数字红点」加到 navigationbar 上，确实不会被 UINavigationTransitionView snapShot 捕捉到，于此也解决了这个矛盾。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)