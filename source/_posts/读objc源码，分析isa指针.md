---
title: 读objc源码，分析isa指针
date: 2021-09-07 03:04
tags: [objc,isa,源码]
categories: [iOS]
---

# 读objc源码，分析isa指针

Tips：本文从objc源码出发分析isa指针的设计逻辑，需要有一定isa指针的基础

说到isa指针，我们脑海中会出现 Class 、objc_class、objc_class 等结构体，甚至会直接回忆起下面这张图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7ek84tsxj60hd0i83za02.jpg)

周末我在学习 objc源码 时发现，如果从数据结构角度出发解析 isa指针，对isa的理解会更深刻，也会觉得isa更有趣。

## 一、源码分析

### （一）NSObject、isa、objc_class 

`NSObject.h`声明`NSObject`结构体如下：

```
@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

说明 `NSObject` 包裹了一个 `isa指针`，这里的 `isa指针` 数据类型为 `Class`，`Class`是被定义的一个关键词：

`typedef struct objc_class *Class;`

`Class`数据结构由 `objc_class`构成，`objc_class`定义如下：

```
struct objc_class : objc_object {
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
}
```

又因为我们已经知道 `Class`实际上就是 `objc_class`构成，所以上面就变成了：

```
struct objc_class : objc_object {
    objc_class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
}
```

也就是说`objc_class`是通过`superclass`串起来的一个链表，也就解释了如下红框部分的含义：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7etitmcpj60i70i5jso02.jpg)

总结下来就是：

`NSObject` 中包含类型为 `objc_class` 的isa指针，而`objc_class`是一个单向链表结构。

说到这里，你可能会产生如下的疑问：

按 objc_class 单向链表的数据结构，怎么解释图中虚线的指向呢？

这是个好问题，我们接着分析。

### （二）objc_object

我们发现，`objc_class`是继承自`objc_object`的，`objc_object`的数据结构如下：

```
struct objc_object {
private:
    isa_t isa;
}
```

`objc_object`中主要的数据结构为`isa_t`,`isa_t`的数据结构如下：

```
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    Class cls;
    uintptr_t bits;
}
```

（union表示这个数据结构是一个联合体，共用内存）

因为 `Class` 实际为 `objc_class`，所以`isa_t`也即：


```
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    objc_class cls;
    uintptr_t bits;
}
```
分析到这里，大家有发现一个神奇的数据结构出现了吗？

>> 0. objc_class 继承自 objc_object
>> 1. objc_object 由 isa_t 数据结构构成，isa_t 数据结构中有一个 objc_class 的字段

`objc_class`是子类，`objc_object`是父类，从设计原则出发，父类是不应该去理解子类的（因为子类可能有很多个），但在我们这个case中，父类竟然还持有了子类，这种设计结构是大苹果有意为之的吗？是为了解决什么问题呢？

于此，我们先暂停3分钟，思考一下下面这两个问题：

- 1. 实例对象和静态类数据结构应该一致吗？
- 2. 实例方法和静态方法怎么存储？

可以这么说，大苹果的isa设计逻辑就是为了解决上面这两个问题。

## 二、设计原则

### （一）实例对象和类对象数据结构应该一致吗？

这个问题可以转换成这样一个问题:我们知道 `objc_class` 是继承 `objc_object`的，那如果我们不声明`objc_object`结构，只用下面 `objc_class`来表示所有对象，有没有问题？

```
struct objc_class {
    objc_class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
    
	// 来自 objc_object 的字段
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    objc_class cls;
    uintptr_t bits;
}
```

够不够用，要看我们对实例对象的需要：
- 1. 调用实例对象的方法
- 2. 找到实例对象的superClass
- 3. 使用实例对象对应类的静态方法 isa_t
- 4. 管理自己的生命周期，引用计数

看起来用 `objc_class`可以实现1、2、3，但4从 objc_class 看起来是无法实现的，所以`objc_object`应运而生。

### （二）isa 和 isa_t

经过上面的分析我们发现 ，isa 和 isa_t 实际上是非常不同的数据结构，isa的结构本质上是 `objc_class`，isa_t则是一个union联合体：

```
struct objc_class : objc_object {
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
}

union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    objc_class cls;
    uintptr_t bits;
}
```

我们来具体看下每个字段是做什么的，先来看 `objc_class`：

- cache_t：方法缓存,优化方法调用的性能
- class_data_bits_t：该类的实例方法链表，指向了类对象的数据区域。在该数据区域内查找相应方法的对应实现

再来看 `isa_t`，这里我借鉴了网上的介绍，原文:[isa_t](https://blog.csdn.net/WangErice/article/details/104801097)

在早期的版本中`objc_object`中isa只是一个Class类型的指针，`Class _Nonnull isa`，也就是说和现在的`objc_class`一样。

在早期的32bit版本中isa就是一个单一的指针,用于存储当前对象的类或者类的元类. 但是在64bit为操作系统上，用一个8字节指针的长度只存储一个对象地址显然是浪费的(操作系统只有一部分地址是可用于存储对象地址的空间)，所以apple对这个isa指针进行了优化.

```
union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    objc_class cls;
    uintptr_t bits;
}
```

作为过渡也为了兼容早期的实现版本，这个结构中保存了变量Class class的实现，同时增加了uintptr_t(unsigned long)类型的变量bits,但由于使用的是联合体(公用体,共用变量空间),所以该结构只占用一个指针的空间.当使用bits变量进行存储时,利用位域结构将变量的各个位进行拆分赋予不同的含义,充分利用了内存空间.

利用位域使得变量内不仅仅保存了指针值，同时还保存了很多有用的信息.

```
# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
#   define ISA_BITFIELD                                                      \
      uintptr_t nonpointer        : 1;                                       \
      uintptr_t has_assoc         : 1;                                       \
      uintptr_t has_cxx_dtor      : 1;                                       \
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ \
      uintptr_t magic             : 6;                                       \
      uintptr_t weakly_referenced : 1;                                       \
      uintptr_t deallocating      : 1;                                       \
      uintptr_t has_sidetable_rc  : 1;                                       \
      uintptr_t extra_rc          : 19
#   define RC_ONE   (1ULL<<45)
#   define RC_HALF  (1ULL<<18)
 
# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x001f800000000001ULL
#   define ISA_MAGIC_VALUE 0x001d800000000001ULL
#   define ISA_BITFIELD                                                        \
      uintptr_t nonpointer        : 1;                                         \
      uintptr_t has_assoc         : 1;                                         \
      uintptr_t has_cxx_dtor      : 1;                                         \
      uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ \
      uintptr_t magic             : 6;                                         \
      uintptr_t weakly_referenced : 1;                                         \
      uintptr_t deallocating      : 1;                                         \
      uintptr_t has_sidetable_rc  : 1;                                         \
      uintptr_t extra_rc          : 8
#   define RC_ONE   (1ULL<<56)
#   define RC_HALF  (1ULL<<7)
 
# else
#   error unknown architecture for packed isa
# endif
```

以主流的arm64为例，主要包含了：

- nonpointer：占用1bit,标识是否开启isa优化.如果是一个指针值该位为0,则表示当前结构的值只是一个指针没有保存其他信息;如果为1,则表示当前结构不是指针,而是一个包含了其他信息的位域结构；
- has_assoc:当前对象是否使用objc_setAssociatedObject动态绑定了额外的属性;
- has_cxx_dtor: 是否含有C++或者OC的析构函数,不包含析构函数时对象释放速度会更快;
- shiftcls:   这个值相当于早期实现中的isa指针，是真实的指针值，在arm64处理器上只占据33位,可见其实在内存中可以用来存储对象指针的空间是很有限的；
- magic：用于判断对象是否已经完成了初始化，在 arm64 中 0x16 是调试器判断当前对象是真的对象还是没有初始化的空间(在 x86_64 中该值为 0x3b);
- weakly_referenced:是否是弱引用对象;
- deallocating:对象是否正在执行析构函数（是否在释放内存）;
- has_sidetable_rc:判断是否需要用sidetable去处理引用计数;
- extra_rc：存储该对象的引用计数值减一后的结果. 当对象的引用计数使用extra_rc足以存储时has_sidetable_rc=0；当对象的引用计数使用extra_rc不能存储时has_sidetable_rc=1.可见对象的引用计数主要存储在两个地方：如果isa中extra_rc足以存储则存储在isa的位域中；如果isa位域不足以存储，就会使用sidetable去存储.


介绍至此，我们应该都明白了 `objc_class`和`objc_object/isa_t`的区别，简单来说，一个主内，一个主外：

- objc_object ：主外，使用isa_t指针承担各种描述性的工作
- objc_class ：主内，承担着方法高效调用和管理的工作

所以，下次如果有人问我们是否理解 isa 吗？我们的第一反应应该是：

你问的是 `isa_t isa `还是 `Class isa`，这两个作用可不一样。

## 三、小结

于此，我们应该很清楚一个实例对象的设计结构了：

实例对象本身的结构是 `objc_object`，它有一个 `isa_t` 结构，负责处理外交和内部关联的工作，`isa_t` 也包含了 `objc_class`。

当我们调用对象的方法，访问对象的内容时，使用的就是`objc_class`结构体，当然了如果你想访问元类对象，也可以使用 `objc_class`中的 `isa_t` 访问 `MetaClass`。

## 四、小练习

最后我们来分析个有趣的题目，来巩固下我们的理解。

### isKindOfClass 和 isMemberOfClass 

首先给出下面的题目：

```
Father为继承自NSObject的类，Son为继承自Father的类：

Son *sonObject = [Son new];    

Father *fatherObject = [Father new];        

BOOL ret1 = [Son isKindOfClass:[fatherObject class]];    

BOOL ret2 = [sonObject isKindOfClass:[Father class]];    

BOOL ret3 = [Son isMemberOfClass:[fatherObject class]];    

BOOL ret4 = [sonObject isMemberOfClass:[Father class]];

NSLog(@"%d------%d-------%d-------%d",ret1,ret2,ret3,ret4);

输出结果如何？

```

这个题目实际考察的是两个方面的问题：
- 实例方法和静态方法的调用链
- isKindOfClass 和 isMemberOfClass 实现的区别

我们首先要清楚的是：

- 调用实例对象的方法，实际上就是调用类对象`objc_class`中的方法
- 调用类的静态方法，实际上就是直接调用类的元类`objc_class`中的方法

明白了这点，我们再看一下 `isKindOfClass` 和 `isMemberOfClass` 具体实现：

```
// isKindOfClass
+ (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = object_getClass((id)self); tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}

- (BOOL)isKindOfClass:(Class)cls {
    for (Class tcls = [self class]; tcls; tcls = tcls->superclass) {
        if (tcls == cls) return YES;
    }
    return NO;
}

// isMemberOfClass
+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}

- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}
```

从源码中可以非常清楚的看到，isKindOfClass 会按 superClass 这条链进行遍历，而 isMemberOfClass 只做一次匹配，也即：

- isKindOfClass : 这个方法用来判断一个对象是否是指定类或者某个从该类继承类的实例对象。
- isMemberOfClass : 这个方法用来判断一个对象是否是指定类的实例对象。

#### 我们分析第一题：

`BOOL ret1 = [Son isKindOfClass:[fatherObject class]];`

`[Son isKindOfClass:]`说明调用的是 `Son`的元类对象的`objc_class`中的方法，但`[fatherObject class]`调用的是`fatherObject`类对象的`objc_class`方法，按照`objc_object`传递流程图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7hmee9q6j60n10nq0uq02.jpg)

`fatherObject`位于 位置③，`Son`初始位于位置②，通过遍历SuperClass走向为：②->④->⑤->⑥，很可惜，一直没有③。

所以第一题结论为 NO。

#### 继而分析第二题：

`BOOL ret2 = [sonObject isKindOfClass:[Father class]];`

首先我们应该一眼就看明白, `sonObject`位于位置①，`Father`位于位置④。

`[sonObject class]` 也即是位置②，继续遍历 superClass，也即位置④，位置匹配上。

所以第二题结论为 YES。

#### 继续分析第三题：

`BOOL ret3 = [Son isMemberOfClass:[fatherObject class]];`

`Son`位于位置②，`fatherObject`位于位置③，`fatherObject class`也即位置④，位置不匹配。

所以第三题结论为 NO。

#### 继续分析第四题：

`BOOL ret4 = [sonObject isMemberOfClass:[Father class]];`

`sonObject`位于位置①，`sonObject class`位于位置②，`Father class`位于位置④，位置不匹配。

阶段性小结：

我们发现无论是 `isKindOfClass` 还是 `isMemberOfClass`，它的判断方式都是在元类阶段，无论你是实例变量还是静态类，做比较时都是在下面绿框范围内：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu7hmup7wvj60nc0njmzc02.jpg)

无非就是 `isKindOfClass` 会通过SuperClass进行遍历，而 `isMemberOfClass `不会遍历。


## 五、总结

明确isa指针可以让我们更好的明白一个对象的生命周期和内在结构，可以说`objc_object`和`objc_class`是构建iOS消息转发机制和Runtime机制的数据结构基石。

我个人倾向于苹果是先完成了`objc_object/class`数据结构，基于此构建了 消息转发和Runtime机制，而不是为了实现消息转发和Runtime机制构建了 `objc_object/class` 数据结构。

面向对象的高级语言，首先对象的设计规则是最重要的，它不仅影响着开发者的使用，还影响着操作系统的设计规范，内存的管理模式。

消息转发和Runtime本质上是技术方案，完成技术方案可以有很多种方式，而因为 `objc_object/class` 的设计特点，大苹果基于此又设计了非常巧妙的 消息转发和Runtime机制 ，这两套机制，我们有空再聊。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)

> 文章首发：http://www.wenwoha.com/?cat_id=5