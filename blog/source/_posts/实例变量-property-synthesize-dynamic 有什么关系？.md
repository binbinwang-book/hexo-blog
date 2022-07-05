---
title: 实例变量/property/synthesize/dynamic 有什么关系？
date: 2021-08-01 22:33
tags: [synthesize]
categories: [iOS]
---

# 实例变量/property/synthesize/dynamic 有什么关系？

很多时候大家直接使用 property，渐渐地都忘记 实例变量、synthesize、dynamic 的使用场景了。

也因为这几个关键词在过去几个版本官方更改过含义，所以有些人新旧版本记混了，但目前版本已经稳定下来了，认认真真对这几个关键词做区分还是非常有必要的。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1fr5f5fuj30dg0773yo.jpg)

## 一、首先我给出两个公式：

```
property == 存取器Accessor（getter/setter） + synthesize(创建实例变量) 

dynamic == !synthesize
```

## 二、不用 property 如何声明 实例变量 的 getter/setter ?

property 实际上是一种生产力的解放，我们先看一下在没有 property 时，如果我们想实现一个实例变量的 setter/getter ，需要怎么写。

```
//  Man.h
#import <Foundation/Foundation.h>

@interface Man : NSObject
{
    // 实例变量
    NSString *name;
    NSString *sex;
}
// setter
- (void)setName:(NSString *)newName;

// getter
- (NSString *)name;

// setter
- (void)setSex:(NSString *)newSex;

// getter
- (NSString *)sex;
@end

//  Man.m
#import "Man.h"

@implementation Man
// setter
- (void)setName:(NSString *)nameTmp
{
    name = nameTmp;
}

// getter
- (NSString *)name
{
    return name;
}

// setter
- (void)setSex:(NSString *)sexTmp
{
    sex = sexTmp;
}

// getter
- (NSString *)sex
{
    return sex;
}
```

完全不需要 synthesize。

而在有了 property 之后，就按如下声明即可：

```
//  Man.h
#import <Foundation/Foundation.h>

@interface Man : NSObject
@property (nonatomic,strong)NSString *name;
@property (nonatomic,strong)NSString *sex;
@end
```

## 三、synthesize 和 dynamic 知识点

### （一）@synthesize知识点

@synthesize name = _name

- _name是成员变量
- name是属性
- 作用是告诉编译器name属性为_name实例变量生成setter and getter方法的实现
- name属性的setter方法是setName,它操作的是_name这个变量。
- 在@synthesize中定义与变量名不同的setter和getter的命名，以此来保护变量不会被不恰当的访问。

@synthesize是为属性添加一个实例变量名，或者说别名。同时会为该属性生成 setter/getter 方法。

如果某属性已经在某处实现了自己的 setter/getter ,可以使用 @dynamic来阻止 @synthesize 自动生成新的 setter/getter 覆盖。

当在 protocol 中声明并实现属性时。协议中声明的属性不会自动生成setter和getter，需要使用@synthesize生成setter和getter。 [UIApplicationDelegate window] 就是个典型的例子。

@property有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize和 @dynamic都没写，那么默认的就是@syntheszie var = _var;

### （二）@dynamic 知识点

@dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）。

假如一个属性被声明为 @dynamic var，然后你没有提供 @setter方法和 @getter 方法。编译的时候没问题，但是当程序运行到 instance.var = someVar，由于缺 setter 方法会导致程序崩溃；

或者当运行到 someVar = var 时，由于缺 getter 方法同样会导致崩溃。

编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

## 四、synthesize 和 dynamic 在有 property 的场景下还有用吗？

我自己在项目中真的是很少再使用 synthesize 和 dynamic，但我经常要review代码，所以还是要搞清楚 synthesize 和 dynamic 的使用场景。

#### （一）synthesize 的使用场景

#### 1. 项目运用

如果我们只是声明了一个 property 的 setter 或者 getter 方法，系统依旧会通过 property 的 synthesize 接口帮我们自动合成对应的 _property ，比如:

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1e49aozuj30im0bot95.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1e4kquutj30iu0dfmxr.jpg)

然而如果我们同时实现了一个 property 的 setter 和 getter 方法，那么系统就不会对这个 property 实行 synthesize 生成 实例变量 了，需要我们手动进行 synthesize，相当于系统默认我们不再使用 property 的自动合成功能，全部改为手动合成：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1e66zu8hj30qt0bfmy0.jpg)

使用 @synthesize 后可以编译通过：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1e6j3uyij30ke0beq3j.jpg)

#### 2. readonly/readwrite 和 synthesize 相关吗？

有关系。

如果我们只声明了一个 property 为 readonly 的，那么系统也会终止 synthesize 合成：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1e93y05mj30i007xq3d.jpg)

但如果我们声明一个 property 是 readwrite 的，那实际上和不声明没有什么区别，所以系统会进行 synthesize 合成的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1ea78teej30i3086dg5.jpg)

如果在 .h 声明了 readonly，在 .m 声明了 readwrite，那么就打破了 @property 本身默认对外暴露 getter/setter 的初衷，那么就需要自己合成 synthesize。

#### 3. 如果使用的是 实例变量，还可以使用 synthesize 吗？

不可以。

通过前面的分析我们明白，synthesize 是关联 property 和 _property 的桥梁，如果我们只声明了实例变量，而未声明 property，那么使用 synthesize 关联是完全没有意义的， 可以说 synthesize 必须和 property 一起使用。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1eond8vbj30oz073aam.jpg)

#### （二）dynamic 的使用场景

我们已经明白： property 声明的属性 sortID 实际上是一个接口，实例变量实际上是 _sortID，是synthesize将 sortID 和 _sortID 关联起来的。

如果我们声明一个 property sortID 只是想实现接口的效果，使用 sortID 的 getter/setter 方法，而并不想 sortID 和 _sortID 进行关联，那么就可以使用 dynamic ，dynamic 使用示例如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1egx23v4j30s90b7mye.jpg)

可能你会觉得，我不使用 @dynamic ，使用 @synthesize 也同样能让编译通过呀：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1ehz8ailj30si0bt0u7.jpg)

说法没错，但这种场景下 property 只是一个接口，如果使用 synthesize 不仅多创建了无用的实例变量 _testObject，而且会让后续接入这块业务的同学十分疑惑：

明明使用 synthesize 关联了 实例变量，但在 getter/setter 方法中又没有使用到，这是闹呢？


最后说一句：

我在查 synthesize 的相关知识点的时候，发现很多文章转来转去，实际上转的内容都是错的，连编译都编译不过。

所以学知识点，还是一定要自己写代码验证一下，不然学到的都是错误的知识，令人作呕，一点都不cool。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)