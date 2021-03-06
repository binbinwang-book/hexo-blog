---
title: 项目中使用Yoga
date: 2022-04-19 20:52
tags: [Yoga]
categories: [iOS]
---


这篇文章介绍了Yoga的使用API，搞懂这篇文章后可以熟练使用yoga进行布局。

对使用Yoga而言，要分清两个概念：方向和容器。

方向指的是：主轴和交叉轴

容器指的是：flex container 和 flex item.

那什么时候应该使用yoga进行布局呢？ 当你需要添加约束时（比如想适配iPad），那么建议你可以使用yoga。

----------------------------



原文链接：https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/#

##### 1. 有帮助的链接

- [Yogalayout](https://yogalayout.com/)
- [Yoga github](https://github.com/facebook/yoga)
- [YogaKit github](https://github.com/facebook/yoga/tree/master/YogaKit)
- [raywenderlich](https://www.raywenderlich.com/530-yoga-tutorial-using-a-cross-platform-layout-engine)
- [Flex Playground](https://codepen.io/enxaneta/full/adLPwv/)

##### 2. 开始前的叨叨

项目中最早用的是Masonry，它用起来确实十分方便，但是它是基于AutoLayout的，所以到了项目中期替换替换成了给予frame的布局方式，那么为什么要在自己的练习项目中使用Yoga呢？理由有如下几方面：

1. Yoga是跨平台的在Android,Reactive Native,iOS 都有对应的版本，这样方便自己后续使用Android,RN时，能够少学点，统一下技术栈（还是懒 ^V^）。
2. Yoga是基于Flex布局的，Flutter,以及Web框架，小程序也都是采用这种布局方式，还是想一劳永逸，并且是基于frame的性能上面是可以接受的，何乐而不为。

这篇主要关注的是Flex 以及 YogaKit，大家如果想了解frame布局，AutoLayout布局原理，以及目前iOS比较主流的布局框架，我后面还会另起一个博客来介绍。

##### 3. Flex布局概念

###### 3.1 Flex 容器布局属性

盒子模型：

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/0000001.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/0000001.png)

```
* position 在当前盒子中item 的定位
* margin item 的外边距。
* border item的边框。
* padding item内边距。
* width & height，当 box-sizing 为 content-box，指内部蓝色的区域。当 box-siziong 为 boder-box时，指含 border 以内的区域。Yoga默认是 border-box。
```

###### * Flex direction

Flex 布局是有两类子对象:flex item 和 flex container。flex item 和 flex container组成一个布局层级树。还包括主轴和交叉轴的概念，要解释主轴和交叉轴的概念必须知道flex direction这个概念。

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/01.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/01.png)

一般如果需要使用flex布局，那么需要使用display属性来开启：

```
display: flex
```

我们先来看下 flex direction及主轴交叉轴：
flex direction 就是布局方向，一般支持row,row reverse,column,clumn reverse. 也就是横轴，横轴逆向，纵轴，纵轴逆向。如果flex direction指定的是row,row reverse 那么主轴就是横轴，如果为column,clumn reverse那么主轴就是纵轴。

```
flex-direction: row / column / row-reverse / column-reverse
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/02.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/02.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/03.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/03.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/04.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/04.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/05.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/05.png)

有了主轴交叉轴的概念后就可以进行后续的概念的介绍了：

###### * Justify-content

Justify-content 是flex item 沿着主轴方向上的布局方式：

flex-start：所有的flex item 沿着主轴布局的开始方向进行布局
flex-end：所有的flex item 从主轴结尾开始沿着主轴逆向布局
center： 所有flex item 居中布局
space-between：所有flex item 在容器内被空白空间均匀间隔开，第一个项目在开端位置，最后一个项目在末端位置。
space-around：所有flex item周围以同等空间均匀间隔，它和space-between的区别是开始和结束有空白区域，并且为中间区域的一半。

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/14.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/14.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/15.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/15.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/16.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/16.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/17.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/17.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/18.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/18.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/19.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/19.png)

###### * Align-items

指定item在交叉轴的对齐方式

```
align-items: stretch / flex-start / flex-end / center / baseline

stretch（默认）每个项目进行拉伸，直到所有item大小占满父容器
flex-start 对齐交叉轴的起点
flex-end 对齐交叉轴的终点
center 交叉轴内居中
baseline 在一行中，所有item以首个item的文字排版为基线对齐，仅在 flex-direction: row / row-reverse 生效
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/09.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/09.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/10.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/10.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/11.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/11.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/12.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/12.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/13.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/13.png)

###### * Flex-wrap

Flex-wrap指明了当可用排版空间不足时，是否允许换行，以及换行后的顺序。

```
flex-wrap: wrap（默认） / nowrap / wrap-reverse

wrap （默认）空间不足时，进行换行
nowrap 不换行
wrap-reverse 换行后，第一行在最下方，行排版方向向上
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/06.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/06.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/07.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/07.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/08.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/08.png)

###### * Align-content

指定container中存在多行情况下，在交叉轴上的布局方式，这里需要注意的是与Align-items的区别，Align-items 是单行上各个元素的对齐方式，是站在item元素的角度，Align-content是多行的情况下，整个内容在交叉轴上的分布关系。是站在交叉轴整体布局角度。

```
align-content: stretch / flex-start / flex-end / center / space-between / space-around

stretch（默认）在交叉轴上的大小进行拉伸，铺满容器
flex-start 行向交叉轴起点对齐
flex-end 行向交叉轴终点对齐
center 行在交叉轴上居中
space-between 均匀排列每一行，第一行放置于起点，最后一行放置于终点
space-around 均匀排列每一行，每一行周围分配相同的空间
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/20.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/20.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/21.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/21.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/22.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/22.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/23.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/23.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/24.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/24.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/25.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/25.png)

###### 3.2 Flex item布局属性

###### * Align-self

align-self 属性可重写flex container 的 align-items 属性。

```
align-self: auto / stretch / flex-start / flex-end / center / base-line
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/26.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/26.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/27.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/27.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/28.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/28.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/29.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/29.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/30.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/30.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/31.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/31.png)
[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/32.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/32.png)

###### * Order

指定项目的排列顺序,值越大排在越后面

```
order: （默认 0）
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/36.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/36.png)

###### * Flex-grow

指定item的放大比例，默认为0，即如果存在剩余空间，也不进行放大。

```
flex-grow: （默认 0）
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/33.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/33.png)

###### * Flex-shrink

指定item的缩小比例，默认为1，即在空间不足（仅当不换行时候起效），所有item等比缩小，当设置为0，该item不进行缩小。

```
flex-shrink: number （默认 1）
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/34.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/34.png)

###### * Flex-basis

指定item的主轴的初始大小，auto 的含义是参考 width 或 height 的大小，

```
flex-basis: number / auto（默认 auto）
```

[![img](https://tbfungeek.github.io/2019/11/05/%E5%9C%A8%E9%A1%B9%E7%9B%AE%E4%B8%AD%E4%BD%BF%E7%94%A8Yoga-%E5%B8%83%E5%B1%80%E5%BC%95%E6%93%8E/35.png)](https://tbfungeek.github.io/2019/11/05/在项目中使用Yoga-布局引擎/35.png)

###### width *height && max-width* max-height && min-width * min-height

item 的尺寸参数，最大尺寸，最小尺寸

###### Aspect Ratio

item 的宽高比

##### 4. YogaKit 使用

YogaKit 有如下属性，有了上面的讲解估计都比较熟悉了

```
@property (nonatomic, readwrite, assign) YGDirection direction;
@property (nonatomic, readwrite, assign) YGFlexDirection flexDirection;
@property (nonatomic, readwrite, assign) YGJustify justifyContent;
@property (nonatomic, readwrite, assign) YGAlign alignContent;
@property (nonatomic, readwrite, assign) YGAlign alignItems;
@property (nonatomic, readwrite, assign) YGAlign alignSelf;
@property (nonatomic, readwrite, assign) YGPositionType position;
@property (nonatomic, readwrite, assign) YGWrap flexWrap;
@property (nonatomic, readwrite, assign) YGOverflow overflow;
@property (nonatomic, readwrite, assign) YGDisplay display;

@property (nonatomic, readwrite, assign) CGFloat flex;
@property (nonatomic, readwrite, assign) CGFloat flexGrow;
@property (nonatomic, readwrite, assign) CGFloat flexShrink;
@property (nonatomic, readwrite, assign) YGValue flexBasis;

@property (nonatomic, readwrite, assign) YGValue left;
@property (nonatomic, readwrite, assign) YGValue top;
@property (nonatomic, readwrite, assign) YGValue right;
@property (nonatomic, readwrite, assign) YGValue bottom;
@property (nonatomic, readwrite, assign) YGValue start;
@property (nonatomic, readwrite, assign) YGValue end;

@property (nonatomic, readwrite, assign) YGValue marginLeft;
@property (nonatomic, readwrite, assign) YGValue marginTop;
@property (nonatomic, readwrite, assign) YGValue marginRight;
@property (nonatomic, readwrite, assign) YGValue marginBottom;
@property (nonatomic, readwrite, assign) YGValue marginStart;
@property (nonatomic, readwrite, assign) YGValue marginEnd;
@property (nonatomic, readwrite, assign) YGValue marginHorizontal;
@property (nonatomic, readwrite, assign) YGValue marginVertical;
@property (nonatomic, readwrite, assign) YGValue margin;

@property (nonatomic, readwrite, assign) YGValue paddingLeft;
@property (nonatomic, readwrite, assign) YGValue paddingTop;
@property (nonatomic, readwrite, assign) YGValue paddingRight;
@property (nonatomic, readwrite, assign) YGValue paddingBottom;
@property (nonatomic, readwrite, assign) YGValue paddingStart;
@property (nonatomic, readwrite, assign) YGValue paddingEnd;
@property (nonatomic, readwrite, assign) YGValue paddingHorizontal;
@property (nonatomic, readwrite, assign) YGValue paddingVertical;
@property (nonatomic, readwrite, assign) YGValue padding;

@property (nonatomic, readwrite, assign) CGFloat borderLeftWidth;
@property (nonatomic, readwrite, assign) CGFloat borderTopWidth;
@property (nonatomic, readwrite, assign) CGFloat borderRightWidth;
@property (nonatomic, readwrite, assign) CGFloat borderBottomWidth;
@property (nonatomic, readwrite, assign) CGFloat borderStartWidth;
@property (nonatomic, readwrite, assign) CGFloat borderEndWidth;
@property (nonatomic, readwrite, assign) CGFloat borderWidth;

@property (nonatomic, readwrite, assign) YGValue width;
@property (nonatomic, readwrite, assign) YGValue height;
@property (nonatomic, readwrite, assign) YGValue minWidth;
@property (nonatomic, readwrite, assign) YGValue minHeight;
@property (nonatomic, readwrite, assign) YGValue maxWidth;
@property (nonatomic, readwrite, assign) YGValue maxHeight;
//只要 width 或者 height 确定，就能确定另外一个变量。
@property (nonatomic, readwrite, assign) CGFloat aspectRatio;
```

我们看下,一个布局实例，整个步骤分成如下4步：

```
1. 设置view的layout
configureLayoutWithBlock:(YGLayoutConfigurationBlock)block
2. 将layout应用到view
applyLayoutPreservingOrigin:(BOOL)preserveOrigin
3. 计算布局
calculateLayoutWithSize
4. 将布局应用到view层级上
YGApplyLayoutToViewHierarchy
```

其中步骤1.和步骤2是我们来完成的，3，4是yoga完成的。

```
-(void)configSubViewsLayout {
    
    [self.view configureLayoutWithBlock:^(YGLayout * layout) {
        layout.isEnabled = YES;
        layout.width = YGPointValue(self.view.bounds.size.width);
        layout.height = YGPointValue(self.view.bounds.size.height);
        layout.alignItems = YGAlignCenter;
    }];
    
    UIView *contentView = [[UIView alloc]init];
    contentView.backgroundColor = [UIColor lightGrayColor];
    [contentView configureLayoutWithBlock:^(YGLayout * layout) {
        layout.isEnabled = true;
        layout.flexDirection =  YGFlexDirectionRow;
        layout.width = YGPointValue(320);
        layout.height = YGPointValue(80);
        layout.marginTop = YGPointValue(100);
        layout.padding =  YGPointValue(10);
    }];
    
    UIView *child1 = [[UIView alloc]init];
    child1.backgroundColor = [UIColor redColor];
    [child1 configureLayoutWithBlock:^(YGLayout * layout) {
        layout.isEnabled = YES;
        layout.width = YGPointValue(80);
        layout.marginRight = YGPointValue(10);
    }];

    UIView *child2 = [[UIView alloc]init];
    child2.backgroundColor = [UIColor blueColor];
    [child2 configureLayoutWithBlock:^(YGLayout * layout) {
        layout.isEnabled = YES;
        layout.width = YGPointValue(80);
        layout.flexGrow = 1;
        layout.height = YGPointValue(20);
        layout.alignSelf = YGAlignCenter;
    }];
    
    [contentView addSubview:child1];
    [contentView addSubview:child2];
    [self.view addSubview:contentView];
}

- (void)layoutSubViews {
    [super layoutSubViews];
    [self.view.yoga applyLayoutPreservingOrigin:YES];
}
```

是不是很像Masonry？ 下一节我们将通过源码分析来看看它具体的工作原理。

##### 5. YogaKit 源码解析

当我们使用view.yoga的时候，YOGA会通过关联属性来为view添加一个YGLayout的属性，这个属性中存放的是yoga的布局约束

```
- (YGLayout *)yoga {
  YGLayout *yoga = objc_getAssociatedObject(self, kYGYogaAssociatedKey);
  if (!yoga) {
    yoga = [[YGLayout alloc] initWithView:self];
    objc_setAssociatedObject(self, kYGYogaAssociatedKey, yoga, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
  }
  return yoga;
}
```

调用 configureLayoutWithBlock的时候，我们会将为当前view添加的yoga关联属性引用传出去，在bock中我们为它设置布局约束。

```
- (void)configureLayoutWithBlock:(YGLayoutConfigurationBlock)block {
  if (block != nil) {
    block(self.yoga);
  }
}
```

我们来看下YGLayout,，它有两个开关属性，isIncludedInLayout，isEnabled一般我们都是需要将其打开的。

```
/**
 进行一次布局运算并更新view的frame。如果不保留原点，根视图的布局结果将会从{0,0}开始
 */
- (void)applyLayoutPreservingOrigin:(BOOL)preserveOrigin;

- (void)applyLayoutPreservingOrigin:(BOOL)preserveOrigin
               dimensionFlexibility:(YGDimensionFlexibility)dimensionFlexibility;
/**
 在没有约束条件的情况下返回view的固有尺寸，这相当于调用了[self sizeThatFits:CGSizeMake(CGFLOAT_MAX, CGFLOAT_MAX)];
 */
@property (nonatomic, readonly, assign) CGSize intrinsicSize;

/**
  根据给定的约束条件，返回view的尺寸
 */
- (CGSize)calculateLayoutWithSize:(CGSize)size;

/**
  返回在使用Flex的子view数量
 */
@property (nonatomic, readonly, assign) NSUInteger numberOfChildren;

/**
 当前视图是否有子视图
 */
@property (nonatomic, readonly, assign) BOOL isLeaf;

/**
 将当前视图是否为脏视图，这些视图将会在下一次布局中进行重新布局
 */
@property (nonatomic, readonly, assign) BOOL isDirty;

/**
 将当前视图标记为脏视图
 */
- (void)markDirty;
```

上面只是通过block将通过关联属性添加的ASLayout类型的yoga属性传递出去，让我们在block里面设置，但是这些属性还没有应用到view上面，所以必须调用applyLayoutPreservingOrigin方法应用这些设置，我们接下来看下applyLayoutPreservingOrigin方法。

```
- (void)applyLayoutPreservingOrigin:(BOOL)preserveOrigin {
  [self calculateLayoutWithSize:self.view.bounds.size];
  YGApplyLayoutToViewHierarchy(self.view, preserveOrigin);
}
```

这里很明确分成两步：

1. measure 指测量item所需要的大小，以及子节点的大小，也就是确定width 和 height
2. layout 指将item确定的放置在具体的 (x, y) 点，也就是确定对应的posion

二者构成了UIView的frame

我们来看下第一步 – 测量

这里首先构建出当前view为起点的节点树，然后通过YGNodeCalculateLayout来算出宽高传递出去。这里关键的是YGNodeCalculateLayout这个方法，它是yoga的算法，是用C++实现的，由于代码很长，其中核心的Flex布局算法实现函数长达几千行，占Yoga.c的 2/3,第一次看很没有勇气看下去。所以后续会另起一篇博客来详细分析Yoga的底层实现。所以这里只要简单知道这些方法是干啥的就好。

```
- (CGSize)calculateLayoutWithSize:(CGSize)size {
    
  NSAssert([NSThread isMainThread], @"Yoga calculation must be done on main.");
  NSAssert(self.isEnabled, @"Yoga is not enabled for this view.");
    
  //构建当前view为起点的节点树
  YGAttachNodesFromViewHierachy(self.view);

  const YGNodeRef node = self.node;
  //计算出节点的宽高
  YGNodeCalculateLayout(
    node,
    size.width /*约束宽度*/,
    size.height /*约束高度*/,
    YGNodeStyleGetDirection(node))/*方向*/;
  //将宽高封装成CGSize 传递出去
  return (CGSize) {
    .width = YGNodeLayoutGetWidth(node),
    .height = YGNodeLayoutGetHeight(node),
  };
}
```

通过上面的计算我们获得了节点的详细参数，所以根据这些参数就可以很容易计算出对应的frame数值。下面是布局的具体代码。

```
static void YGApplyLayoutToViewHierarchy(UIView *view, BOOL preserveOrigin)
{
    
  NSCAssert([NSThread isMainThread], @"Framesetting should only be done on the main thread.");

  const YGLayout *yoga = view.yoga;

  if (!yoga.isIncludedInLayout) {
     return;
  }

  YGNodeRef node = yoga.node;
    
  //获得左上角的坐标
  const CGPoint topLeft = {
    YGNodeLayoutGetLeft(node),
    YGNodeLayoutGetTop(node),
  };

 //计算右下角的坐标
  const CGPoint bottomRight = {
    topLeft.x + YGNodeLayoutGetWidth(node),
    topLeft.y + YGNodeLayoutGetHeight(node),
  };
  
  //如果preserveOrigin为true那么会保留原来的节点坐标
  const CGPoint origin = preserveOrigin ? view.frame.origin : CGPointZero;
  //布局该节点
  view.frame = (CGRect) {
    .origin = {
      .x = YGRoundPixelValue(topLeft.x + origin.x),
      .y = YGRoundPixelValue(topLeft.y + origin.y),
    },
    .size = {
      .width = YGRoundPixelValue(bottomRight.x) - YGRoundPixelValue(topLeft.x),
      .height = YGRoundPixelValue(bottomRight.y) - YGRoundPixelValue(topLeft.y),
    },
  };

  if (!yoga.isLeaf) {
    //对子节点进行布局
    for (NSUInteger i=0; i<view.subviews.count; i++) {
      YGApplyLayoutToViewHierarchy(view.subviews[i], NO);
    }
  }
}
```

到目前位置我们分析了整个YogaKit的源码，其实最核心最重要的还是Yoga底层的代码，这里由于篇幅原因将其放到下一篇博客中。