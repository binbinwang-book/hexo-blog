---
title: 架构学习自省与Runtime深析
date: 2021-08-00 01:02
tags: [Runtime,自省]
categories: [iOS]
---

# 架构学习自省与Runtime深析

## 前言

Runtime机制我自己是学了很多遍，然而每次用到的时候还是Google搜索然后回忆知识点，今天我在思考一个问题，为什么我学Runtime这种架构性的知识点那么费劲，反思出了学习系统知识点的方法。

首先反思自己之前学 Runtime 都有什么样的特点？

我之前基本都是Google runtime机制，然后看源码进行学习，类似这种应付别人询问或大学里面对考试的心态。

这种学习心态有个最大的问题，就是无法 学以致用 ，也就是学到的知识点不能内化，不能用在自己实际开发工程上，如果只是考试，比如学习洛伦磁力这种知识，那么只理解原理，不用理解应用也可以，但在计算机领域上，要坚守这句话：

`What I cannot create, I do not understand.`

所以学习计算机知识，一定要按照开发需求的流程去接触，要真正可以用代码实现它，如果自己不能用代码实现它，要说明原因：

- 0. 打算做一个什么样的效果，为什么需要需要做，不做行不行？
- 1. 怎么做？方案有哪几种？为什么选这种方案
- 2. 做起来会有什么问题，为什么自己做不出来，别人能做出来？自己还差了哪些？

## 一、为什么要做 Runtime机制？

CTO：Hi，兄弟们，我们要搞一套 Runtime机制

Dev：？？ 什么是 Runtime机制，搞Runtime干什么？是为了解决什么问题？C语言做不了吗？

CTO：现在技术圈面向一个新语言开发有 静态 和 动态 两种选项，动态语言 的功能和扩展性更强，未来如果想让开发者使用我们的编程语言，动态语言一定更受欢迎。

Dev：动态语言是吧，这个我知道，你是想做到哪个程度的动态？是数据类型全部动态检查还是打算怎么搞？

CTO：我们要做到开发者无感知的动态，也即编译期并不能决定真正调用哪个函数，只有在真正运行时才会根据函数的名称找到对应的函数来调用。

Dev：我去... 那得做两个东西，一个是 `编译器` ，另一个是`提前设定好的逻辑`，可以让系统在 runtime 期间能自己根据设定好的逻辑找到要调用的方法。

CTO：没错！`编译器`那块你先不用担心，已经有隔壁组启动了 clang 的开发，你这边可以开始着手研究`提前设定好的逻辑`这块，暂且我们把它称为`Runtime system`吧。

Dev：收到，我思考一下实现这个需求的数据结构和思路，稍后发送一份技术方案给你。

```
To CTO："Runtime System"数据结构和思路构想

	既然要实现更灵活的消息传递机制，但我们不能无节制的让response和call关联混乱。
	
	所以从数据结构上我主张按`面向对象`的实现方式，允许 response 在发起 call 的`继承链`中检索响应方法。
	（`继承链`中的每一层级都持有一个isa指针，通过isa指针将对象串联起来。）
	
	此外，也额外做一套不按`继承链`响应的消息转发机制，支持开发者在特殊场景下使用。
```

Tips:所以到这里大家要明白，`runtime system`是一个逻辑写死的处理器，作用是：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9k8hjbf8j30ch02iwef.jpg)

在系统中调用某个方法，经过 `Runtime system`设定的规则后，可以找到应该响应的方法。

## 二、isa 和 class

source：

[isa和class](https://halfrost.com/objc_runtime_isa_class/)


![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9pczsuo8j30ku0rsgol.jpg)

### （一）runtime的起源

Runtime 又叫运行时，是一套底层的 C 语言 API，是 iOS 系统的核心之一。开发者在编码过程中，可以给任意一个对象发送消息，在编译阶段只是确定了要向接收者发送这条消息，而接受者将要如何响应和处理这条消息，那就要看运行时来决定了。

C语言中，在编译期，函数的调用就会决定调用哪个函数。
而OC的函数，属于动态调用过程，在编译期并不能决定真正调用哪个函数，只有在真正运行时才会根据函数的名称找到对应的函数来调用。

Objective-C 是一个动态语言，这意味着它不仅需要一个`编译器`，也需要一个`运行时系统`来动态得创建类和对象、进行消息传递和转发。

### （二）调用 runtime 的三个层级

Objc 在三种层面上与 Runtime 系统进行交互：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9ped8y0dj31840oymza.jpg)

#### 1. 过 Objective-C 源代码

一般情况开发者只需要编写 OC 代码即可，Runtime 系统自动在幕后把我们写的源代码在编译阶段转换成运行时代码，在运行时确定对应的数据结构和调用具体哪个方法。

#### 2. 通过 Foundation 框架的 NSObject 类定义的方法

在OC的世界中，除了NSProxy类以外，所有的类都是NSObject的子类。在Foundation框架下，NSObject和NSProxy两个基类，定义了类层次结构中该类下方所有类的公共接口和行为。NSProxy是专门用于实现代理对象的类，这个类暂时本篇文章不提。这两个类都遵循了NSObject协议。在NSObject协议中，声明了所有OC对象的公共方法。

在NSObject协议中，有以下5个方法，是可以从Runtime中获取信息，让对象进行自我检查。

```
- (Class)class OBJC_SWIFT_UNAVAILABLE("use 'anObject.dynamicType' instead");
- (BOOL)isKindOfClass:(Class)aClass;
- (BOOL)isMemberOfClass:(Class)aClass;
- (BOOL)conformsToProtocol:(Protocol *)aProtocol;
- (BOOL)respondsToSelector:(SEL)aSelector;
```

- class方法返回对象的类；
- isKindOfClass: 和 -isMemberOfClass: 方法检查对象是否存在于指定的类的继承体系中；
- respondsToSelector: 检查对象能否响应指定的消息；
- conformsToProtocol:检查对象是否实现了指定协议类的方法；

在NSObject的类中还定义了一个方法

`- (IMP)methodForSelector:(SEL)aSelector;`

这个方法会返回指定方法实现的地址IMP。

以上这些方法会在本篇文章中详细分析具体实现。

#### 3. 通过对 Runtime 库函数的直接调用

关于库函数可以在Objective-C Runtime Reference中查看 Runtime 函数的详细文档。

关于这一点，其实还有一个小插曲。当我们导入了objc/Runtime.h和objc/message.h两个头文件之后，我们查找到了Runtime的函数之后，代码打完，发现没有代码提示了，那些函数里面的参数和描述都没有了。对于熟悉Runtime的开发者来说，这并没有什么难的，因为参数早已铭记于胸。但是对于新手来说，这是相当不友好的。而且，如果是从iOS6开始开发的同学，依稀可能能感受到，关于Runtime的具体实现的官方文档越来越少了？可能还怀疑是不是错觉。其实从Xcode5开始，苹果就不建议我们手动调用Runtime的API，也同样希望我们不要知道具体底层实现。所以IDE上面默认代了一个参数，禁止了Runtime的代码提示，源码和文档方面也删除了一些解释。

具体设置如下:

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9pi2x36tj31920843za.jpg)

如果发现导入了两个库文件之后，仍然没有代码提示，就需要把这里的设置改成NO，即可。

### （三）NSObject起源

由上面一章节，我们知道了与Runtime交互有3种方式，前两种方式都与NSObject有关，那我们就从NSObject基类开始说起。

以下源码分析均来自objc4-680

NSObject的定义如下

```
typedef struct objc_class *Class;

@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}
```

在Objc2.0之前，objc_class源码如下：

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
    
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif
    
} OBJC2_UNAVAILABLE;
```

NSObject -> Class isa -> objc_class 

在这里可以看到，在一个类中，有超类的指针，类名，版本的信息。
ivars是objc_ivar_list成员变量列表的指针；methodLists是指向objc_method_list指针的指针。*methodLists是指向方法列表的指针。这里如果动态修改*methodLists的值来添加成员方法，这也是Category实现的原理，同样解释了Category不能添加属性的原因。

然后在2006年苹果发布Objc 2.0之后，objc_class的定义就变成下面这个样子了。

```
typedef struct objc_class *Class;
typedef struct objc_object *id;

@interface Object { 
    Class isa; 
}

@interface NSObject <NSObject> {
    Class isa  OBJC_ISA_AVAILABILITY;
}

struct objc_object {
private:
    isa_t isa;
}

struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
}

union isa_t 
{
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    Class cls;
    uintptr_t bits;
}

```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9pn99axqj31h40tqwha.jpg)

把源码的定义转化成类图，就是上图的样子。

从上述源码中，我们可以看到，Objective-C 对象都是 C 语言结构体实现的，在objc2.0中，所有的对象都会包含一个isa_t类型的结构体。

objc_object被源码typedef成了id类型，这也就是我们平时遇到的id类型。这个结构体中就只包含了一个isa_t类型的结构体。这个结构体在下面会详细分析。

objc_class继承于objc_object。所以在objc_class中也会包含isa_t类型的结构体isa。至此，可以得出结论：Objective-C 中类也是一个对象。

在 objc_class 中，有一共有4个成员：
- isa
- 父类的指针
- 方法缓存
- 该类的实例方法链表

object类和NSObject类里面分别都包含一个objc_class类型的isa。

上图的左半边类的关系描述完了，接着先从isa来说起。

当一个对象的实例方法被调用的时候，会通过isa找到相应的类，然后在该类的class_data_bits_t中去查找方法。class_data_bits_t是指向了类对象的数据区域。在该数据区域内查找相应方法的对应实现。

但是在我们调用类方法的时候，类对象的isa里面是什么呢？这里为了和对象查找方法的机制一致，遂引入了元类(meta-class)的概念。

在引入元类之后，类对象和对象查找方法的机制就完全统一了。

对象的实例方法调用时，通过对象的 isa 在类中获取方法的实现。
类对象的类方法调用时，通过类的 isa 在元类中获取方法的实现。

meta-class之所以重要，是因为它存储着一个类的所有类方法。每个类都会有一个单独的meta-class，因为每个类的类方法基本不可能完全相同。

对应关系的图如下图，下图很好的描述了对象，类，元类之间的关系:

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9pqozd8rj30hd0i8752.jpg)

图中实线是 super_class指针，虚线是isa指针。

- 1. Root class (class)其实就是NSObject，NSObject是没有超类的，所以Root class(class)的superclass指向nil。
- 2. 每个Class都有一个isa指针指向唯一的Meta class
- 3. Root class(meta)的superclass指向Root class(class)，也就是NSObject，形成一个回路。
- 4. 每个Meta class的isa指针都指向Root class (meta)。

我们其实应该明白，类对象和元类对象是唯一的，对象是可以在运行时创建无数个的。而在main方法执行之前，从 dyld到runtime这期间，类对象和元类对象在这期间被创建。

#### 1. isa_t结构体的具体实现

接下来我们就该研究研究isa的具体实现了。objc_object里面的isa是isa_t类型。通过查看源码，我们可以知道isa_t是一个union联合体。

```
struct objc_object {
private:
    isa_t isa;
public:
    // initIsa() should be used to init the isa of new objects only.
    // If this object already has an isa, use changeIsa() for correctness.
    // initInstanceIsa(): objects with no custom RR/AWZ
    void initIsa(Class cls /*indexed=false*/);
    void initInstanceIsa(Class cls, bool hasCxxDtor);
private:
    void initIsa(Class newCls, bool indexed, bool hasCxxDtor);
｝
```

那就从initIsa方法开始研究。下面以arm64为例。

```
inline void
objc_object::initInstanceIsa(Class cls, bool hasCxxDtor)
{
    initIsa(cls, true, hasCxxDtor);
}

inline void
objc_object::initIsa(Class cls, bool indexed, bool hasCxxDtor)
{
    if (!indexed) {
        isa.cls = cls;
    } else {
        isa.bits = ISA_MAGIC_VALUE;
        isa.has_cxx_dtor = hasCxxDtor;
        isa.shiftcls = (uintptr_t)cls >> 3;
    }
}
```

initIsa第二个参数传入了一个true，所以initIsa就会执行else里面的语句。

```
# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33; // MACH_VM_MAX_ADDRESS 0x1000000000
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 19;
#       define RC_ONE   (1ULL<<45)
#       define RC_HALF  (1ULL<<18)
    };

# elif __x86_64__
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x001f800000000001ULL
#   define ISA_MAGIC_VALUE 0x001d800000000001ULL
    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 44; // MACH_VM_MAX_ADDRESS 0x7fffffe00000
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 8;
#       define RC_ONE   (1ULL<<56)
#       define RC_HALF  (1ULL<<7)
    };

```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9q3uqqv6j31au0ts0v4.jpg)

ISA_MAGIC_VALUE = 0x000001a000000001ULL转换成二进制是11010000000000000000000000000000000000001，结构如下图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9q45g65jj31a20e6dhe.jpg)

关于参数的说明：

第一位index，代表是否开启isa指针优化。index = 1，代表开启isa指针优化。

在2013年9月，苹果推出了iPhone5s，与此同时，iPhone5s配备了首个采用64位架构的A7双核处理器，为了节省内存和提高执行效率，苹果提出了Tagged Pointer的概念。对于64位程序，引入Tagged Pointer后，相关逻辑能减少一半的内存占用，以及3倍的访问速度提升，100倍的创建、销毁速度提升。

在WWDC2013的《Session 404 Advanced in Objective-C》视频中，苹果介绍了 Tagged Pointer。 Tagged Pointer的存在主要是为了节省内存。我们知道，对象的指针大小一般是与机器字长有关，在32位系统中，一个指针的大小是32位（4字节），而在64位系统中，一个指针的大小将是64位（8字节）。

假设我们要存储一个NSNumber对象，其值是一个整数。正常情况下，如果这个整数只是一个NSInteger的普通变量，那么它所占用的内存是与CPU的位数有关，在32位CPU下占4个字节，在64位CPU下是占8个字节的。而指针类型的大小通常也是与CPU位数相关，一个指针所占用的内存在32位CPU下为4个字节，在64位CPU下也是8个字节。如果没有Tagged Pointer对象，从32位机器迁移到64位机器中后，虽然逻辑没有任何变化，但这种NSNumber、NSDate一类的对象所占用的内存会翻倍。如下图所示：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9q5n6xpnj30mj0f2t9k.jpg)

苹果提出了Tagged Pointer对象。由于NSNumber、NSDate一类的变量本身的值需要占用的内存大小常常不需要8个字节，拿整数来说，4个字节所能表示的有符号整数就可以达到20多亿（注：2^31=2147483648，另外1位作为符号位)，对于绝大多数情况都是可以处理的。所以，引入了Tagged Pointer对象之后，64位CPU下NSNumber的内存图变成了以下这样：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9q75fhb2j30md0fawf9.jpg)

- has_assoc

对象含有或者曾经含有关联引用，没有关联引用的可以更快地释放内存

- has_cxx_dtor

表示该对象是否有 C++ 或者 Objc 的析构器

- shiftcls

类的指针。arm64架构中有33位可以存储类指针。

源码中isa.shiftcls = (uintptr_t)cls >> 3;
将当前地址右移三位的主要原因是用于将 Class 指针中无用的后三位清除减小内存的消耗，因为类的指针要按照字节（8 bits）对齐内存，其指针后三位都是没有意义的 0。具体可以看[从 NSObject 的初始化了解 isa](https://github.com/draveness/analyze/blob/master/contents/objc/%E4%BB%8E%20NSObject%20%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E4%BA%86%E8%A7%A3%20isa.md#shiftcls)这篇文章里面的shiftcls分析。

- magic

判断对象是否初始化完成，在arm64中0x16是调试器判断当前对象是真的对象还是没有初始化的空间。

- weakly_referenced

对象被指向或者曾经指向一个 ARC 的弱变量，没有弱引用的对象可以更快释放

- deallocating

对象是否正在释放内存

- has_sidetable_rc

判断该对象的引用计数是否过大，如果过大则需要其他散列表来进行存储。

- extra_rc

存放该对象的引用计数值减一后的结果。对象的引用计数超过 1，会存在这个这个里面，如果引用计数为 10，extra_rc的值就为 9。

ISA_MAGIC_MASK 和 ISA_MASK 分别是通过掩码的方式获取MAGIC值 和 isa类指针。

Tips :掩码是一串二进制代码对目标字段进行位与运算，屏蔽当前的输入位。

```
inline Class 
objc_object::ISA() 
{
    assert(!isTaggedPointer()); 
    return (Class)(isa.bits & ISA_MASK);
}
```

#### 2. cache_t的具体实现

还是继续看源码

```
struct cache_t {
    struct bucket_t *_buckets;
    mask_t _mask;
    mask_t _occupied;
}

typedef unsigned int uint32_t;
typedef uint32_t mask_t;  // x86_64 & arm64 asm are less efficient with 16-bits

typedef unsigned long  uintptr_t;
typedef uintptr_t cache_key_t;

struct bucket_t {
private:
    cache_key_t _key;
    IMP _imp;
}
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9qcsm2e6j30rc0dqt93.jpg)

根据源码，我们可以知道cache_t中存储了一个bucket_t的结构体，和两个unsigned int的变量。

- mask：分配用来缓存bucket的总数。
- occupied：表明目前实际占用的缓存bucket的个数。

bucket_t的结构体中存储了一个unsigned long和一个IMP。IMP是一个函数指针，指向了一个方法的具体实现。

cache_t中的bucket_t *_buckets其实就是一个散列表，用来存储Method的链表。

Cache的作用主要是为了优化方法调用的性能。当对象receiver调用方法message时，首先根据对象receiver的isa指针查找到它对应的类，然后在类的methodLists中搜索方法，如果没有找到，就使用super_class指针到父类中的methodLists查找，一旦找到就调用方法。如果没有找到，有可能消息转发，也可能忽略它。但这样查找方式效率太低，因为往往一个类大概只有20%的方法经常被调用，占总调用次数的80%。所以使用Cache来缓存经常调用的方法，当调用方法时，优先在Cache查找，如果没有找到，再到methodLists查找。

#### 3. class_data_bits_t的具体实现

源码实现如下：

```
struct class_data_bits_t {

    // Values are the FAST_ flags above.
    uintptr_t bits;
}

struct class_rw_t {
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;

    Class firstSubclass;
    Class nextSiblingClass;

    char *demangledName;
}

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;
    
    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};

```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9qembvxpj31a20todi1.jpg)

在 objc_class结构体中的注释写到 class_data_bits_t相当于 class_rw_t指针加上 rr/alloc 的标志。

```
class_data_bits_t bits; // class_data_bits_t == (class_rw_t *) + custom rr/alloc flags
```

它为我们提供了便捷方法用于返回其中的 class_rw_t *指针：

```
class_rw_t *data() {
    return bits.data();
}

```

在编译期类的结构中的 class_data_bits_t *data指向的是一个 class_ro_t *指针：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9qi0tvs5j30hm0modgt.jpg)

在运行时调用 realizeClass方法，会做以下3件事情：

- 1. 从 class_data_bits_t调用 data方法，将结果从 class_rw_t强制转换为 class_ro_t指针
- 2. 初始化一个 class_rw_t结构体
- 3. 设置结构体 ro的值以及 flag

最后调用methodizeClass方法，把类里面的属性，协议，方法都加载进来。

然后聊一下 method。

```
struct method_t {
    SEL name;
    const char *types;
    IMP imp;

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```

方法method的定义如上。里面包含3个成员变量。SEL是方法的名字name。types是Type Encoding类型编码，类型可参考Type Encoding，在此不细说。

Type Encodings 作为 方法三大组成要素之一，还是很重要的。


|  编码  |  意义  | 
|  ----  | ---- |
|c	|char 类型|
|i|	int 类型|
|s|	short 类型|
|l|	long 类型，仅用在 32-bit 设备上|
|q|	long long 类型|
|C|	unsigned char 类型|
|I|	unsigned int 类型|
|S|	unsigned short 类型|
|L|	unsigned long 类型|
|Q|	unsigned long long 类型|
|f|	float 类型|
|d|	double 类型，long double 不被 ObjC 支持，所以也是指向此编码|
|B|	bool 或 _Bool 类型|
|v|	void 类型|
|*|	C 字串（char *）类型|
|@|	对象（id）类型|
|#|	Class 类型|
|:|	SEL 类型|
|[array type]	|C 数组类型（注意这不是 NSArray）|
|{name=type…}|	结构体类型|
|(name=type…)	|联合体类型|
|bnum|	位段（bit field）类型用 b 表示，num 表示字节数，这个类型很少用|
|^type|	一个指向 type 类型的指针类型|
|?|	未知类型|

IMP是一个函数指针，指向的是函数的具体实现。`在runtime中消息传递和转发的目的就是为了找到IMP`，并执行函数。

整个运行时过程可以描述如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9ql4kvxaj30ro0imdhe.jpg)

更加详细的分析，请看@Draveness 的这篇文章[深入解析 ObjC 中方法的结构](https://github.com/draveness/analyze/blob/master/contents/objc/%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90%20ObjC%20%E4%B8%AD%E6%96%B9%E6%B3%95%E7%9A%84%E7%BB%93%E6%9E%84.md#%E6%B7%B1%E5%85%A5%E8%A7%A3%E6%9E%90-objc-%E4%B8%AD%E6%96%B9%E6%B3%95%E7%9A%84%E7%BB%93%E6%9E%84)

到此，总结一下objc_class 1.0和2.0的差别。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9qlpqaq5j312n0u0dib.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9qlzq2h8j312n0u077e.jpg)


## 三、消息发送与转发

source：

[消息发送与转发](https://halfrost.com/objc_runtime_objc_msgsend/)

现在越来越多的app都使用了JSPatch实现app热修复，而JSPatch 能做到通过 JS 调用和改写 OC 方法最根本的原因是 Objective-C 是动态语言，OC 上所有方法的调用/类的生成都通过 Objective-C Runtime 在运行时进行，我们可以通过类名/方法名反射得到相应的类和方法，也可以替换某个类的方法为新的实现，理论上你可以在运行时通过类名/方法名调用到任何 OC 方法，替换任何类的实现以及新增任意类。今天就来详细解析一下OC中runtime最为吸引人的地方。

### （一）objc_msgSend 函数简介

最初接触到OC Runtime，一定是从[receiver message]这里开始的。[receiver message]会被编译器转化为：

```
id objc_msgSend ( id self, SEL op, ... );
```

这是一个可变参数函数。第二个参数类型是SEL。SEL在OC中是selector方法选择器。

```
typedef struct objc_selector *SEL;

```

objc_selector是一个映射到方法的`C字符串`。需要注意的是@selector()选择子只与`函数名`有关。不同类中相同名字的方法所对应的方法选择器是相同的，即使方法名字相同而变量类型不同也会导致它们具有相同的方法选择器。由于这点特性，也导致了OC不支持函数重载。

在receiver拿到对应的selector之后，如果自己无法执行这个方法，那么该条消息要被转发。或者临时动态的添加方法实现。如果转发到最后依旧没法处理，程序就会崩溃。

`所以编译期仅仅是确定了要发送消息，而消息如何处理是要运行期需要解决的事情。`

objc_msgSend函数究竟会干什么事情呢？

```
1. Check for ignored selectors (GC) and short-circuit.
 2. Check for nil target.
    If nil & nil receiver handler configured, jump to handler
    If nil & no handler (default), cleanup and return.
 3. Search the class’s method cache for the method IMP(use hash to find&store method in cache)
    -1. If found, jump to it.
    -2. Not found: lookup the method IMP in the class itself corresponding its hierarchy chain.
        If found, load it into cache and jump to it.
        If not found, jump to forwarding mechanism.
```

总结一下objc_msgSend会做一下几件事情：

- 1.检测这个 selector是不是要忽略的。
- 2.检查target是不是为nil。
	- 如果这里有相应的nil的处理函数，就跳转到相应的函数中。
	- 如果没有处理nil的函数，就自动清理现场并返回。这一点就是为何在OC中给nil发送消息不会崩溃的原因。
- 3.确定不是给nil发消息之后，在该class的缓存中查找方法对应的IMP实现。
	- 如果找到，就跳转进去执行。
	- 如果没有找到，就在方法分发表里面继续查找，一直找到NSObject为止。
- 4.如果还没有找到，那就需要开始消息转发阶段了。至此，发送消息Messaging阶段完成。这一阶段主要完成的是通过select()快速查找IMP的过程。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9rwsgqurj30ig0u8t9z.jpg)


### （二）消息发送Messaging阶段—objc_msgSend源码解析

在这篇文章Obj-C Optimization: The faster objc_msgSend中看到了这样一段C版本的objc_msgSend的源码。

```
#include <objc/objc-runtime.h>

id  c_objc_msgSend( struct objc_class /* ahem */ *self, SEL _cmd, ...)
{
   struct objc_class    *cls;
   struct objc_cache    *cache;
   unsigned int         hash;
   struct objc_method   *method;   
   unsigned int         index;
   
   if( self)
   {
      cls   = self->isa;
      cache = cls->cache;
      hash  = cache->mask;
      index = (unsigned int) _cmd & hash;
      
      do
      {
         method = cache->buckets[ index];
         if( ! method)
            goto recache;
         index = (index + 1) & cache->mask;
      }
      while( method->method_name != _cmd);
      return( (*method->method_imp)( (id) self, _cmd));
   }
   return( (id) self);

recache:
   /* ... */
   return( 0);
}
```

该源码中有一个do-while循环，这个循环就是上一章里面提到的在方法分发表里面查找method的过程。

不过在obj4-680里面的objc-msg-x86_64.s文件中实现是一段汇编代码。

```
/********************************************************************
 *
 * id objc_msgSend(id self, SEL _cmd,...);
 *
 ********************************************************************/
 
 .data
 .align 3
 .globl _objc_debug_taggedpointer_classes
_objc_debug_taggedpointer_classes:
 .fill 16, 8, 0

 ENTRY _objc_msgSend
 MESSENGER_START

 NilTest NORMAL

 GetIsaFast NORMAL  // r11 = self->isa
 CacheLookup NORMAL  // calls IMP on success

 NilTestSupport NORMAL

 GetIsaSupport NORMAL

// cache miss: go search the method lists
LCacheMiss:
 // isa still in r11
 MethodTableLookup %a1, %a2 // r11 = IMP
 cmp %r11, %r11  // set eq (nonstret) for forwarding
 jmp *%r11   // goto *imp

 END_ENTRY _objc_msgSend

 
 ENTRY _objc_msgSend_fixup
 int3
 END_ENTRY _objc_msgSend_fixup

 
 STATIC_ENTRY _objc_msgSend_fixedup
 // Load _cmd from the message_ref
 movq 8(%a2), %a2
 jmp _objc_msgSend
 END_ENTRY _objc_msgSend_fixedup

```

来分析一下这段汇编代码。

乍一看，如果从LCacheMiss:这里上下分开，可以很明显的看到objc_msgSend就干了两件事情—— CacheLookup 和 MethodTableLookup。

```
/////////////////////////////////////////////////////////////////////
//
// NilTest return-type
//
// Takes: $0 = NORMAL or FPRET or FP2RET or STRET
//  %a1 or %a2 (STRET) = receiver
//
// On exit:  Loads non-nil receiver in %a1 or %a2 (STRET), or returns zero.
//
/////////////////////////////////////////////////////////////////////

.macro NilTest
.if $0 == SUPER  ||  $0 == SUPER_STRET
 error super dispatch does not test for nil
.endif

.if $0 != STRET
 testq %a1, %a1
.else
 testq %a2, %a2
.endif
 PN
 jz LNilTestSlow_f
.endmacro

```

NilTest是用来检测是否为nil的。传入参数有4种，NORMAL / FPRET / FP2RET / STRET。

- objc_msgSend 传入的参数是NilTest NORMAL
- objc_msgSend_fpret 传入的参数是NilTest FPRET
- objc_msgSend_fp2ret 传入的参数是NilTest FP2RET
- objc_msgSend_stret 传入的参数是NilTest STRET

如果检测方法的接受者是nil，那么系统会自动clean并且return。

GetIsaFast宏可以快速地获取到对象的 isa 指针地址（放到 r11 寄存器，r10会被重写；在 arm 架构上是直接赋值到 r9）

```
.macro CacheLookup
 
 ldrh r12, [r9, #CACHE_MASK] // r12 = mask
 ldr r9, [r9, #CACHE] // r9 = buckets
.if $0 == STRET  ||  $0 == SUPER_STRET
 and r12, r12, r2  // r12 = index = SEL & mask
.else
 and r12, r12, r1  // r12 = index = SEL & mask
.endif
 add r9, r9, r12, LSL #3 // r9 = bucket = buckets+index*8
 ldr r12, [r9]  // r12 = bucket->sel
2:
.if $0 == STRET  ||  $0 == SUPER_STRET
 teq r12, r2
.else
 teq r12, r1
.endif
 bne 1f
 CacheHit $0
1: 
 cmp r12, #1
 blo LCacheMiss_f  // if (bucket->sel == 0) cache miss
 it eq   // if (bucket->sel == 1) cache wrap
 ldreq r9, [r9, #4]  // bucket->imp is before first bucket
 ldr r12, [r9, #8]!  // r12 = (++bucket)->sel
 b 2b

.endmacro
```

r12里面存的是方法method，r9里面是cache。r1，r2是SEL。在这个CacheLookup函数中，不断的通过SEL与cache中的bucket->sel进行比较，如果r12 = = 0，则跳转到LCacheMiss_f标记去继续执行。如果r12找到了,r12 = =1，即在cache中找到了相应的SEL，则直接执行该IMP(放在r10中)。

程序跳到LCacheMiss，就说明cache中无缓存，未命中缓存。这个时候就要开始下一阶段MethodTableLookup的查找了。

```
/////////////////////////////////////////////////////////////////////
//
// MethodTableLookup classRegister, selectorRegister
//
// Takes: $0 = class to search (a1 or a2 or r10 ONLY)
//  $1 = selector to search for (a2 or a3 ONLY)
//   r11 = class to search
//
// On exit: imp in %r11
//
/////////////////////////////////////////////////////////////////////
.macro MethodTableLookup

 MESSENGER_END_SLOW
 
 SaveRegisters

 // _class_lookupMethodAndLoadCache3(receiver, selector, class)

 movq $0, %a1
 movq $1, %a2
 movq %r11, %a3
 call __class_lookupMethodAndLoadCache3

 // IMP is now in %rax
 movq %rax, %r11

 RestoreRegisters

.endmacro

```

MethodTableLookup 可以算是个接口层宏，主要用于保存环境与准备参数，来调用 __class_lookupMethodAndLoadCache3函数（在objc-class.mm中）。具体是把receiver, selector, class三个参数传给$0，$1，r11，然后再去调用lookupMethodAndLoadCache3方法。最后会将 IMP 返回（从 r11 挪到 rax）。最后在 objc_msgSend中调用 IMP。

```
/***********************************************************************
* _class_lookupMethodAndLoadCache.
* Method lookup for dispatchers ONLY. OTHER CODE SHOULD USE lookUpImp().
* This lookup avoids optimistic cache scan because the dispatcher 
* already tried that.
**********************************************************************/
IMP _class_lookupMethodAndLoadCache3(id obj, SEL sel, Class cls)
{        
    return lookUpImpOrForward(cls, sel, obj, 
                              YES/*initialize*/, NO/*cache*/, YES/*resolver*/);
}
```

__class_lookupMethodAndLoadCache3函数也是个接口层（C编写），此函数提供相应参数配置，实际功能在lookUpImpOrForward函数中。

再来看看lookUpImpOrForward函数实现

```
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver)
{
    Class curClass;
    IMP imp = nil;
    Method meth;
    bool triedResolver = NO;

    /*
    中间是查找过程，详细解析见下。
    */

    // paranoia: look for ignored selectors with non-ignored implementations
    assert(!(ignoreSelector(sel)  &&  imp != (IMP)&_objc_ignored_method));

    // paranoia: never let uncached leak out
    assert(imp != _objc_msgSend_uncached_impcache);

    return imp;
}
```

接下来一行行的解析。

```
runtimeLock.assertUnlocked();
```

runtimeLock.assertUnlocked(); 这个是加一个读写锁，保证线程安全。

```
    // Optimistic cache lookup
    if (cache) {
        imp = cache_getImp(cls, sel);
        if (imp) return imp;
    }
```

lookUpImpOrForward第5个新参是是否找到cache的布尔量，如果传入的是YES，那么就会调用cache_getImp方法去找到缓存里面的IMP。

```
/********************************************************************
 * IMP cache_getImp(Class cls, SEL sel)
 *
 * On entry: a1 = class whose cache is to be searched
 *  a2 = selector to search for
 *
 * If found, returns method implementation.
 * If not found, returns NULL.
 ********************************************************************/

 STATIC_ENTRY _cache_getImp

// do lookup
 movq %a1, %r11  // move class to r11 for CacheLookup
 CacheLookup GETIMP  // returns IMP on success

LCacheMiss:
// cache miss, return nil
 xorl %eax, %eax
 ret

LGetImpExit:
 END_ENTRY  _cache_getImp

```

cache_getImp会把找到的IMP放在r11中。

```
    if (!cls->isRealized()) {
        rwlock_writer_t lock(runtimeLock);
        realizeClass(cls);
    }
```

调用realizeClass方法是申请class_rw_t的可读写空间。

```
    if (initialize  &&  !cls->isInitialized()) {
        _class_initialize (_class_getNonMetaClass(cls, inst));
    }

```

_class_initialize是类初始化的过程。

```
retry:
    runtimeLock.read();
```

runtimeLock.read();这里加了一个读锁。因为在运行时中会动态的添加方法，为了保证线程安全，所以要加锁。从这里开始，下面会出现5处goto done的地方，和一处goto retry。

```
 done:
    runtimeLock.unlockRead();
```

在done的地方，会完成IMP的查找，于是可以打开读锁。

```
    // Ignore GC selectors
    if (ignoreSelector(sel)) {
        imp = _objc_ignored_method;
        cache_fill(cls, sel, imp, inst);
        goto done;
    }
```

紧接着GC selectors是为了忽略macOS中GC垃圾回收机制用到的方法，iOS则没有这一步。如果忽略，则进行cache_fill，然后跳转到goto done那里去。

```
void cache_fill(Class cls, SEL sel, IMP imp, id receiver)
{
#if !DEBUG_TASK_THREADS
    mutex_locker_t lock(cacheUpdateLock);
    cache_fill_nolock(cls, sel, imp, receiver);
#else
    _collecting_in_critical();
    return;
#endif
}


static void cache_fill_nolock(Class cls, SEL sel, IMP imp, id receiver)
{
    cacheUpdateLock.assertLocked();

    // Never cache before +initialize is done
    if (!cls->isInitialized()) return;

    // Make sure the entry wasn't added to the cache by some other thread 
    // before we grabbed the cacheUpdateLock.
    if (cache_getImp(cls, sel)) return;

    cache_t *cache = getCache(cls);
    cache_key_t key = getKey(sel);

    // Use the cache as-is if it is less than 3/4 full
    mask_t newOccupied = cache->occupied() + 1;
    mask_t capacity = cache->capacity();
    if (cache->isConstantEmptyCache()) {
        // Cache is read-only. Replace it.
        cache->reallocate(capacity, capacity ?: INIT_CACHE_SIZE);
    }
    else if (newOccupied <= capacity / 4 * 3) {
        // Cache is less than 3/4 full. Use it as-is.
    }
    else {
        // Cache is too full. Expand it.
        cache->expand();
    }
    bucket_t *bucket = cache->find(key, receiver);
    if (bucket->key() == 0) cache->incrementOccupied();
    bucket->set(key, imp);
}

```

在cache_fill中还会去调用cache_fill_nolock函数，如果缓存中的内容大于容量的 3/4就会扩充缓存，使缓存的大小翻倍。找到第一个空的 bucket_t，以 (SEL, IMP)的形式填充进去。

```
    // Try this class's cache.

    imp = cache_getImp(cls, sel);
    if (imp) goto done;

```

如果不忽略，则再次尝试从类的cache中获取IMP，如果获取到，然后也会跳转到goto done去。

```
    // Try this class's method lists.

    meth = getMethodNoSuper_nolock(cls, sel);
    if (meth) {
        log_and_fill_cache(cls, meth->imp, sel, inst, cls);
        imp = meth->imp;
        goto done;
    }
```

如果在cache缓存中获取失败，则再去类方法列表里面进行查找。找到后跳转到goto done。

```
    // Try superclass caches and method lists.

    curClass = cls;
    while ((curClass = curClass->superclass)) {
        // Superclass cache.
        imp = cache_getImp(curClass, sel);
        if (imp) {
            if (imp != (IMP)_objc_msgForward_impcache) {
                // Found the method in a superclass. Cache it in this class.
                log_and_fill_cache(cls, imp, sel, inst, curClass);
                goto done;
            }
            else {
                // Found a forward:: entry in a superclass.
                // Stop searching, but don't cache yet; call method 
                // resolver for this class first.
                break;
            }
        }

```

如果以上尝试都失败了，接下来就会循环尝试父类的缓存和方法列表。一直找到NSObject为止。因为NSObject的superclass为nil，才跳出循环。

如果在父类中找到了该方法method的IMP，接下来就应该把这个方法cache回自己的缓存中。fill完之后跳转goto done语句。

```
        // Superclass method list.
        meth = getMethodNoSuper_nolock(curClass, sel);
        if (meth) {
            log_and_fill_cache(cls, meth->imp, sel, inst, curClass);
            imp = meth->imp;
            goto done;
        }
    }

```

如果没有在父类的cache中找到IMP，继续在父类的方法列表里面查找。如果找到，跳转goto done语句。

```
static method_t * getMethodNoSuper_nolock(Class cls, SEL sel)
{
    runtimeLock.assertLocked();

    assert(cls->isRealized());
    // fixme nil cls? 
    // fixme nil sel?

    for (auto mlists = cls->data()->methods.beginLists(), 
              end = cls->data()->methods.endLists(); 
         mlists != end;
         ++mlists)
    {
        method_t *m = search_method_list(*mlists, sel);
        if (m) return m;
    }

    return nil;
}

```

这里可以解析一下method的查找过程。在getMethodNoSuper_nolock方法中，会遍历一次methodList链表，从begin一直遍历到end。遍历过程中会调用search_method_list函数。

```
static method_t *search_method_list(const method_list_t *mlist, SEL sel)
{
    int methodListIsFixedUp = mlist->isFixedUp();
    int methodListHasExpectedSize = mlist->entsize() == sizeof(method_t);
    
    if (__builtin_expect(methodListIsFixedUp && methodListHasExpectedSize, 1)) {
        return findMethodInSortedMethodList(sel, mlist);
    } else {
        // Linear search of unsorted method list
        for (auto& meth : *mlist) {
            if (meth.name == sel) return &meth;
        }
    }

#if DEBUG
    // sanity-check negative results
    if (mlist->isFixedUp()) {
        for (auto& meth : *mlist) {
            if (meth.name == sel) {
                _objc_fatal("linear search worked when binary search did not");
            }
        }
    }
#endif

    return nil;
}

```

在search_method_list函数中，会去判断当前methodList是否有序，如果有序，会调用findMethodInSortedMethodList方法，这个方法里面的实现是一个二分搜索，具体代码就不贴了。如果非有序，就调用线性的傻瓜式遍历搜索。

```
    // No implementation found. Try method resolver once.

    if (resolver  &&  !triedResolver) {
        runtimeLock.unlockRead();
        _class_resolveMethod(cls, sel, inst);
        // Don't cache the result; we don't hold the lock so it may have 
        // changed already. Re-do the search from scratch instead.
        triedResolver = YES;
        goto retry;
    }

```

如果父类找到NSObject还没有找到，那么就会开始尝试_class_resolveMethod方法。注意，这些需要打开读锁，因为开发者可能会在这里动态增加方法实现，所以不需要缓存结果。此处虽然锁被打开，可能会出现线程问题，所以在执行完_class_resolveMethod方法之后，会goto retry，重新执行一遍之前查找的过程。

```
/***********************************************************************
* _class_resolveMethod
* Call +resolveClassMethod or +resolveInstanceMethod.
* Returns nothing; any result would be potentially out-of-date already.
* Does not check if the method already exists.
**********************************************************************/
void _class_resolveMethod(Class cls, SEL sel, id inst)
{
    if (! cls->isMetaClass()) {
        // try [cls resolveInstanceMethod:sel]
        _class_resolveInstanceMethod(cls, sel, inst);
    } 
    else {
        // try [nonMetaClass resolveClassMethod:sel]
        // and [cls resolveInstanceMethod:sel]
        _class_resolveClassMethod(cls, sel, inst);
        if (!lookUpImpOrNil(cls, sel, inst, 
                            NO/*initialize*/, YES/*cache*/, NO/*resolver*/)) 
        {
            _class_resolveInstanceMethod(cls, sel, inst);
        }
    }
}

```

这个函数首先判断是否是meta-class类，如果不是元类，就执行_class_resolveInstanceMethod，如果是元类，执行_class_resolveClassMethod。这里有一个lookUpImpOrNil的函数调用。

```

IMP lookUpImpOrNil(Class cls, SEL sel, id inst, 
                   bool initialize, bool cache, bool resolver)
{
    IMP imp = lookUpImpOrForward(cls, sel, inst, initialize, cache, resolver);
    if (imp == _objc_msgForward_impcache) return nil;
    else return imp;
}

```

在这个函数实现中，还会去调用lookUpImpOrForward去查找有没有传入的sel的实现，但是返回值还会返回nil。在imp == _objc_msgForward_impcache会返回nil。_objc_msgForward_impcache是一个标记，这个标记用来表示在父类的缓存中停止继续查找。

```

IMP class_getMethodImplementation(Class cls, SEL sel)
{
    IMP imp;

    if (!cls  ||  !sel) return nil;

    imp = lookUpImpOrNil(cls, sel, nil, 
                         YES/*initialize*/, YES/*cache*/, YES/*resolver*/);

    // Translate forwarding function to C-callable external version
    if (!imp) {
        return _objc_msgForward;
    }

    return imp;
}
```

再回到_class_resolveMethod的实现中，如果lookUpImpOrNil返回nil，就代表在父类中的缓存中找到，于是需要再调用一次_class_resolveInstanceMethod方法。保证给sel添加上了对应的IMP。

```
    // No implementation found, and method resolver didn't help. 
    // Use forwarding.

    imp = (IMP)_objc_msgForward_impcache;
    cache_fill(cls, sel, imp, inst);

```

回到lookUpImpOrForward方法中，如果也没有找到IMP的实现，那么method resolver也没用了，只能进入消息转发阶段。进入这个阶段之前，imp变成_objc_msgForward_impcache。最后再加入缓存中。


### （三）消息转发Message Forwarding阶段

到了转发阶段，会调用id _objc_msgForward(id self, SEL _cmd,...)方法。在objc-msg-x86_64.s中有其汇编的实现。

``` STATIC_ENTRY __objc_msgForward_impcache
 // Method cache version

 // THIS IS NOT A CALLABLE C FUNCTION
 // Out-of-band condition register is NE for stret, EQ otherwise.

 MESSENGER_START
 nop
 MESSENGER_END_SLOW
 
 jne __objc_msgForward_stret
 jmp __objc_msgForward

 END_ENTRY __objc_msgForward_impcache
 
 
 ENTRY __objc_msgForward
 // Non-stret version

 movq __objc_forward_handler(%rip), %r11
 jmp *%r11

 END_ENTRY __objc_msgForward
```

在执行_objc_msgForward之后会调用__objc_forward_handler函数。

```
// Default forward handler halts the process.
__attribute__((noreturn)) void objc_defaultForwardHandler(id self, SEL sel)
{
    _objc_fatal("%c[%s %s]: unrecognized selector sent to instance %p "
                "(no message forward handler is installed)", 
                class_isMetaClass(object_getClass(self)) ? '+' : '-', 
                object_getClassName(self), sel_getName(sel), self);
}
```

在最新的Objc2.0中会有一个objc_defaultForwardHandler，看源码实现我们可以看到熟悉的语句。当我们给一个对象发送一个没有实现的方法的时候，如果其父类也没有这个方法，则会崩溃，报错信息类似于这样：unrecognized selector sent to instance，然后接着会跳出一些堆栈信息。这些信息就是从这里而来。

```
void *_objc_forward_handler = (void*)objc_defaultForwardHandler;

#if SUPPORT_STRET
struct stret { int i[100]; };
__attribute__((noreturn)) struct stret objc_defaultForwardStretHandler(id self, SEL sel)
{
    objc_defaultForwardHandler(self, sel);
}
void *_objc_forward_stret_handler = (void*)objc_defaultForwardStretHandler;
#endif

#endif

void objc_setForwardHandler(void *fwd, void *fwd_stret)
{
    _objc_forward_handler = fwd;
#if SUPPORT_STRET
    _objc_forward_stret_handler = fwd_stret;
#endif
}

```

要设置转发只要重写_objc_forward_handler方法即可。在objc_setForwardHandler方法中，可以设置ForwardHandler。

但是当你想要弄清objc_setForwardHandler调用栈的情况的时候，你会发现打印不出来入口。因为苹果在这里做了点手脚。关于objc_setForwardHandler的调用，以及之后的消息转发调用栈的问题，需要用到逆向的知识。推荐大家看这两篇文章就会明白其中的原理。

Objective-C 消息发送与转发机制原理
Hmmm, What’s that Selector?

还是回到消息转发上面来。当前的SEL无法找到相应的IMP的时候，开发者可以通过重写- (id)forwardingTargetForSelector:(SEL)aSelector方法来“偷梁换柱”，把消息的接受者换成一个可以处理该消息的对象。

```
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(Method:)){
        return otherObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}

```

当然也可以替换类方法，那就要重写 + (id)forwardingTargetForSelector:(SEL)aSelector方法，返回值是一个类对象。

```
+ (id)forwardingTargetForSelector:(SEL)aSelector {
    if(aSelector == @selector(xxx)) {
        return NSClassFromString(@"Class name");
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

这一步是替消息找备援接收者，如果这一步返回的是nil，那么补救措施就完全的失效了，Runtime系统会向对象发送methodSignatureForSelector:消息，并取到返回的方法签名用于生成NSInvocation对象。为接下来的完整的消息转发生成一个 NSMethodSignature对象。NSMethodSignature 对象会被包装成 NSInvocation 对象，forwardInvocation: 方法里就可以对 NSInvocation 进行处理了。

接下来未识别的方法崩溃之前，系统会做一次完整的消息转发。

我们只需要重写下面这个方法，就可以自定义我们自己的转发逻辑了。

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
         [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}

```

实现此方法之后，若发现某调用不应由本类处理，则会调用超类的同名方法。如此，继承体系中的每个类都有机会处理该方法调用的请求，一直到NSObject根类。如果到NSObject也不能处理该条消息，那么就是再无挽救措施了，只能抛出“doesNotRecognizeSelector”异常了。

至此，消息发送和转发的过程都清楚明白了。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9t2laqe9j30yz0u0ju3.jpg)

### （四）forwardInvocation 实现 AOP 的例子

这里我想举一个好玩的例子，来说明一下forwardInvocation的使用方法。

这个例子中我们会利用runtime消息转发机制创建一个动态代理。利用这个动态代理来转发消息。这里我们会用到两个基类的另外一个神秘的类，NSProxy。

NSProxy类和NSObject同为OC里面的基类，但是NSProxy类是一种抽象的基类，无法直接实例化，可用于实现代理模式。它通过实现一组经过简化的方法，代替目标对象捕捉和处理所有的消息。NSProxy类也同样实现了NSObject的协议声明的方法，而且它有两个必须实现的方法。

```
- (void)forwardInvocation:(NSInvocation *)invocation;
- (nullable NSMethodSignature *)methodSignatureForSelector:(SEL)sel NS_SWIFT_UNAVAILABLE("NSInvocation and related APIs not available");
```

另外还需要说明的是，NSProxy类的子类必须声明并实现至少一个init方法，这样才能符合OC中创建和初始化对象的惯例。Foundation框架里面也含有多个NSProxy类的具体实现类，比如：

- NSDistantObject类：定义其他应用程序或线程中对象的代理类。
- NSProtocolChecker类：定义对象，使用这话对象可以限定哪些消息能够发送给另外一个对象。

接下来就来看看下面这个好玩的例子。

```
#import <Foundation/Foundation.h>

@interface Student : NSObject
-(void)study:(NSString *)subject andRead:(NSString *)bookName;
-(void)study:(NSString *)subject :(NSString *)bookName;
@end
```

定义一个student类，里面随便给两个方法。

```
#import "Student.h"
#import <objc/runtime.h>

@implementation Student

-(void)study:(NSString *)subject :(NSString *)bookName
{
    NSLog(@"Invorking method on %@ object with selector %@",[self class],NSStringFromSelector(_cmd));
}

-(void)study:(NSString *)subject andRead:(NSString *)bookName
{
    NSLog(@"Invorking method on %@ object with selector %@",[self class],NSStringFromSelector(_cmd));
}
@end

```

在两个方法实现里面增加log信息，这是为了一会打印的时候方便知道调用了哪个方法。

```
#import <Foundation/Foundation.h>
#import "Invoker.h"

@interface AspectProxy : NSProxy

/** 通过NSProxy实例转发消息的真正对象 */
@property(strong) id proxyTarget;
/** 能够实现横切功能的类（遵守Invoker协议）的实例 */
@property(strong) id<Invoker> invoker;
/** 定义了哪些消息会调用横切功能 */
@property(readonly) NSMutableArray *selectors;

// AspectProxy类实例的初始化方法
- (id)initWithObject:(id)object andInvoker:(id<Invoker>)invoker;
- (id)initWithObject:(id)object selectors:(NSArray *)selectors andInvoker:(id<Invoker>)invoker;
// 向当前的选择器列表中添加选择器
- (void)registerSelector:(SEL)selector;

@end

```

定义一个AspectProxy类，这个类专门用来转发消息的。

```
#import "AspectProxy.h"

@implementation AspectProxy

- (id)initWithObject:(id)object selectors:(NSArray *)selectors andInvoker:(id<Invoker>)invoker{
    _proxyTarget = object;
    _invoker = invoker;
    _selectors = [selectors mutableCopy];
    
    return self;
}

- (id)initWithObject:(id)object andInvoker:(id<Invoker>)invoker{
    return [self initWithObject:object selectors:nil andInvoker:invoker];
}

// 添加另外一个选择器
- (void)registerSelector:(SEL)selector{
    NSValue *selValue = [NSValue valueWithPointer:selector];
    [self.selectors addObject:selValue];
}

// 为目标对象中被调用的方法返回一个NSMethodSignature实例
// 运行时系统要求在执行标准转发时实现这个方法
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel{
    return [self.proxyTarget methodSignatureForSelector:sel];
}

/**
 *  当调用目标方法的选择器与在AspectProxy对象中注册的选择器匹配时，forwardInvocation:会
 *  调用目标对象中的方法，并根据条件语句的判断结果调用AOP（面向切面编程）功能
 */
- (void)forwardInvocation:(NSInvocation *)invocation{
    // 在调用目标方法前执行横切功能
    if ([self.invoker respondsToSelector:@selector(preInvoke:withTarget:)]) {
        if (self.selectors != nil) {
            SEL methodSel = [invocation selector];
            for (NSValue *selValue in self.selectors) {
                if (methodSel == [selValue pointerValue]) {
                    [[self invoker] preInvoke:invocation withTarget:self.proxyTarget];
                    break;
                }
            }
        }else{
            [[self invoker] preInvoke:invocation withTarget:self.proxyTarget];
        }
    }
    
    // 调用目标方法
    [invocation invokeWithTarget:self.proxyTarget];
    
    // 在调用目标方法后执行横切功能
    if ([self.invoker respondsToSelector:@selector(postInvoke:withTarget:)]) {
        if (self.selectors != nil) {
            SEL methodSel = [invocation selector];
            for (NSValue *selValue in self.selectors) {
                if (methodSel == [selValue pointerValue]) {
                    [[self invoker] postInvoke:invocation withTarget:self.proxyTarget];
                    break;
                }
            }
        }else{
            [[self invoker] postInvoke:invocation withTarget:self.proxyTarget];
        }
    }
}
```

接着我们定义一个代理协议

```
#import <Foundation/Foundation.h>

@protocol Invoker <NSObject>

@required
// 在调用对象中的方法前执行对功能的横切
- (void)preInvoke:(NSInvocation *)inv withTarget:(id)target;
@optional
// 在调用对象中的方法后执行对功能的横切
- (void)postInvoke:(NSInvocation *)inv withTarget:(id)target;

@end
```

最后还需要一个遵守协议的类

```
#import <Foundation/Foundation.h>
#import "Invoker.h"

@interface AuditingInvoker : NSObject<Invoker>//遵守Invoker协议
@end


#import "AuditingInvoker.h"

@implementation AuditingInvoker

- (void)preInvoke:(NSInvocation *)inv withTarget:(id)target{
    NSLog(@"before sending message with selector %@ to %@ object", NSStringFromSelector([inv selector]),[target className]);
}
- (void)postInvoke:(NSInvocation *)inv withTarget:(id)target{
    NSLog(@"after sending message with selector %@ to %@ object", NSStringFromSelector([inv selector]),[target className]);

}
@end
```

在这个遵循代理类里面我们只实现协议里面的两个方法。

写出测试代码

```
#import <Foundation/Foundation.h>
#import "AspectProxy.h"
#import "AuditingInvoker.h"
#import "Student.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        id student = [[Student alloc] init];

        // 设置代理中注册的选择器数组
        NSValue *selValue1 = [NSValue valueWithPointer:@selector(study:andRead:)];
        NSArray *selValues = @[selValue1];
        // 创建AuditingInvoker
        AuditingInvoker *invoker = [[AuditingInvoker alloc] init];
        // 创建Student对象的代理studentProxy
        id studentProxy = [[AspectProxy alloc] initWithObject:student selectors:selValues andInvoker:invoker];
        
        // 使用指定的选择器向该代理发送消息---例子1
        [studentProxy study:@"Computer" andRead:@"Algorithm"];
        
        // 使用还未注册到代理中的其他选择器，向这个代理发送消息！---例子2
        [studentProxy study:@"mathematics" :@"higher mathematics"];
        
        // 为这个代理注册一个选择器并再次向其发送消息---例子3
        [studentProxy registerSelector:@selector(study::)];
        [studentProxy study:@"mathematics" :@"higher mathematics"];
    }
    return 0;
}
```

这里有3个例子。里面会分别输出什么呢？

```
before sending message with selector study:andRead: to Student object
Invorking method on Student object with selector study:andRead:
after sending message with selector study:andRead: to Student object

Invorking method on Student object with selector study::

before sending message with selector study:: to Student object
Invorking method on Student object with selector study::
after sending message with selector study:: to Student object

```

例子1中会输出3句话。调用Student对象的代理中的study:andRead:方法，会使该代理调用AuditingInvoker对象中的preInvoker:方法、真正目标（Student对象）中的study:andRead:方法，以及AuditingInvoker对象中的postInvoker:方法。一个方法的调用，调用起了3个方法。原因是study:andRead:方法是通过Student对象的代理注册的；

例子2就只会输出1句话。调用Student对象代理中的study::方法，因为该方法还未通过这个代理注册，所以程序仅会将调用该方法的消息转发给Student对象，而不会调用AuditorInvoker方法。

例子3又会输出3句话了。因为study::通过这个代理进行了注册，然后程序再次调用它，在这次调用过程中，程序会调用AuditingInvoker对象中的AOP方法和真正目标（Student对象）中的study::方法。

这个例子就实现了一个简单的AOP(Aspect Oriented Programming)面向切面编程。我们把一切功能"切"出去，与其他部分分开，这样可以提高程序的模块化程度。AOP能解耦也能动态组装，可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能。比如上面的例子三，我们通过把方法注册到动态代理类中，于是就实现了该类也能处理方法的功能。

### （五）Runtime中的优化

关于Runtime系统中，有3种地方进行了优化。

- 1.方法列表的缓存
- 2.虚函数表vTable
- 3.dyld共享缓存

#### 1.方法列表的缓存

在消息发送过程中，查找IMP的过程，会优先查找缓存。这个缓存会存储最近使用过的方法都缓存起来。这个cache和CPU里面的cache的工作方式有点类似。原理是调用的方法有可能经常会被调用。如果没有这个缓存，直接去类方法的方法链表里面去查找，查询效率实在太低。所以查找IMP会优先搜索饭方法缓存，如果没有找到，接着会在虚函数表中寻找IMP。如果找到了，就会把这个IMP存储到缓存中备用。

基于这个设计，使Runtime系统能能够执行快速高效的方法查询操作。

#### 2.虚函数表

虚函数表也称为分派表，是编程语言中常用的动态绑定支持机制。在OC的Runtime运行时系统库实现了一种自定义的虚函数表分派机制。这个表是专门用来提高性能和灵活性的。这个虚函数表是用来存储IMP类型的数组。每个object-class都有这样一个指向虚函数表的指针。

#### 3.dyld共享缓存

在我们的程序中，一定会有很多自定义类，而这些类中，很多SEL是重名的，比如alloc，init等等。Runtime系统需要为每一个方法给定一个SEL指针，然后为每次调用个各个方法更新元数据，以获取唯一值。这个过程是在应用程序启动的时候完成。为了提高这一部分的执行效率，Runtime会通过dyld共享缓存实现选择器的唯一性。

dyld是一种系统服务，用于定位和加载动态库。它含有共享缓存，能够使多个进程共用这些动态库。dyld共享缓存中含有一个选择器表，从而能使运行时系统能够通过使用缓存访问共享库和自定义类的选择器。

关于dyld的知识可以看看这篇文章[dyld: Dynamic Linking On OS X](https://www.mikeash.com/pyblog/friday-qa-2012-11-09-dyld-dynamic-linking-on-os-x.html)

## 四、如何正确使用 Runtime

source：

[如何正确使用 Runtime](https://halfrost.com/how_to_use_runtime/)

### （一）Runtime 的优点

#### 1. 实现多继承Multiple Inheritance

在上一篇文章里面讲到的forwardingTargetForSelector:方法就能知道，一个类可以做到继承多个类的效果，只需要在这一步将消息转发给正确的类对象就可以模拟多继承的效果。

在官方文档上记录了这样一段例子。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9tnyuhl7j30ag06pmx6.jpg)

在OC程序中可以借用消息转发机制来实现多继承的功能。 在上图中，一个对象对一个消息做出回应，类似于另一个对象中的方法借过来或是“继承”过来一样。 在图中，warrior实例转发了一个negotiate消息到Diplomat实例中，执行Diplomat中的negotiate方法，结果看起来像是warrior实例执行了一个和Diplomat实例一样的negotiate方法，其实执行者还是Diplomat实例。

这使得不同继承体系分支下的两个类可以“继承”对方的方法，这样一个类可以响应自己继承分支里面的方法，同时也能响应其他不相干类发过来的消息。在上图中Warrior和Diplomat没有继承关系，但是Warrior将negotiate消息转发给了Diplomat后，就好似Diplomat是Warrior的超类一样。

消息转发提供了许多类似于多继承的特性，但是他们之间有一个很大的不同：

多继承：合并了不同的行为特征在一个单独的对象中，会得到一个重量级多层面的对象。

消息转发：将各个功能分散到不同的对象中，得到的一些轻量级的对象，这些对象通过消息通过消息转发联合起来。

这里值得说明的一点是，即使我们利用转发消息来实现了“假”继承，但是NSObject类还是会将两者区分开。像respondsToSelector:和 isKindOfClass:这类方法只会考虑继承体系，不会考虑转发链。比如上图中一个Warrior对象如果被问到是否能响应negotiate消息：

```
if ( [aWarrior respondsToSelector:@selector(negotiate)] )
```

结果是NO，虽然它能够响应negotiate消息而不报错，但是它是靠转发消息给Diplomat类来响应消息的。

如果非要制造假象，反应出这种“假”的继承关系，那么需要重新实现 respondsToSelector:和 isKindOfClass:来加入你的转发算法：

```
- (BOOL)respondsToSelector:(SEL)aSelector
{
    if ( [super respondsToSelector:aSelector] )
        return YES;
    else {
        /* Here, test whether the aSelector message can     *
         * be forwarded to another object and whether that  *
         * object can respond to it. Return YES if it can.  */
    }
    return NO;
}

```

除了respondsToSelector:和 isKindOfClass:之外，instancesRespondToSelector:中也应该写一份转发算法。如果使用了协议，conformsToProtocol:也一样需要重写。类似地，如果一个对象转发它接受的任何远程消息，它得给出一个methodSignatureForSelector:来返回准确的方法描述，这个方法会最终响应被转发的消息。比如一个对象能给它的替代者对象转发消息，它需要像下面这样实现methodSignatureForSelector:

```
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (!signature) {
        signature = [surrogate methodSignatureForSelector:selector];
    }
    return signature;
}

```

需要引起注意的一点，实现methodSignatureForSelector方法是一种先进的技术，只适用于没有其他解决方案的情况下。它不会作为继承的替代。如果您必须使用这种技术，请确保您完全理解类做的转发和您转发的类的行为。请勿滥用！


#### 2. Method Swizzling

提到Objective-C 中的 Runtime，大多数人第一个想到的可能就是黑魔法Method Swizzling。毕竟这是Runtime里面很强大的一部分，它可以通过Runtime的API实现更改任意的方法，理论上可以在运行时通过类名/方法名hook到任何 OC 方法，替换任何类的实现以及新增任意类。

举的最多的例子应该就是埋点统计用户信息的例子。

假设我们需要在页面上不同的地方统计用户信息，常见做法有两种：

- 1. 傻瓜式的在所有需要统计的页面都加上代码。这样做简单，但是重复的代码太多。
- 2. 把统计的代码写入基类中，比如说BaseViewController。这样虽然代码只需要写一次，但是UITableViewController，UICollectionViewcontroller都需要写一遍，这样重复的代码依旧不少。

基于这两点，我们这时候选用Method Swizzling来解决这个事情最优雅。

##### (1)Method Swizzling原理

Method Swizzing是发生在运行时的，主要用于在运行时将两个Method进行交换，我们可以将Method Swizzling代码写到任何地方，但是只有在这段Method Swilzzling代码执行完毕之后互换才起作用。而且Method Swizzling也是iOS中AOP(面相切面编程)的一种实现方式，我们可以利用苹果这一特性来实现AOP编程。

Method Swizzling本质上就是对IMP和SEL进行交换。

##### (2)Method Swizzling使用

一般我们使用都是新建一个分类，在分类中进行Method Swizzling方法的交换。交换的代码模板如下：

```
#import <objc/runtime.h>
@implementation UIViewController (Swizzling)
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);
        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        BOOL didAddMethod = class_addMethod(class,
                                            originalSelector,
                                            method_getImplementation(swizzledMethod),
                                            method_getTypeEncoding(swizzledMethod));
        if (didAddMethod) {
            class_replaceMethod(class,
                                swizzledSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}
#pragma mark - Method Swizzling
- (void)xxx_viewWillAppear:(BOOL)animated {
    [self xxx_viewWillAppear:animated];
    NSLog(@"viewWillAppear: %@", self);
}
@end

```

Method Swizzling可以在运行时通过修改类的方法列表中selector对应的函数或者设置交换方法实现，来动态修改方法。可以重写某个方法而不用继承，同时还可以调用原先的实现。所以通常应用于在category中添加一个方法。

##### (3)Method Swizzling注意点

a.Swizzling应该总在+load中执行

Objective-C在运行时会自动调用类的两个方法+load和+initialize。+load会在类初始加载时调用， +initialize方法是以懒加载的方式被调用的，如果程序一直没有给某个类或它的子类发送消息，那么这个类的 +initialize方法是永远不会被调用的。所以Swizzling要是写在+initialize方法中，是有可能永远都不被执行。

和+initialize比较+load能保证在类的初始化过程中被加载。

关于+load和+initialize的比较可以参看这篇文章《Objective-C +load vs +initialize》

b.Swizzling应该总是在dispatch_once中执行

Swizzling会改变全局状态，所以在运行时采取一些预防措施，使用dispatch_once就能够确保代码不管有多少线程都只被执行一次。这将成为Method Swizzling的最佳实践。

这里有一个很容易犯的错误，那就是继承中用了Swizzling。如果不写dispatch_once就会导致Swizzling失效！

举个例子，比如同时对NSArray和NSMutableArray中的objectAtIndex:方法都进行了Swizzling，这样可能会导致NSArray中的Swizzling失效的。

可是为什么会这样呢？
原因是，我们没有用dispatch_once控制Swizzling只执行一次。如果这段Swizzling被执行多次，经过多次的交换IMP和SEL之后，结果可能就是未交换之前的状态。

比如说父类A的B方法和子类C的D方法进行交换，交换一次后，父类A持有D方法的IMP，子类C持有B方法的IMP，但是再次交换一次，就又还原了。父类A还是持有B方法的IMP，子类C还是持有D方法的IMP，这样就相当于咩有交换。可以看出，如果不写dispatch_once，偶数次交换以后，相当于没有交换，Swizzling失效！

c.Swizzling在+load中执行时，不要调用[super load]

原因同注意点二，如果是多继承，并且对同一个方法都进行了Swizzling，那么调用[super load]以后，父类的Swizzling就失效了。

d.上述模板中没有错误

有些人怀疑我上述给的模板可能有错误。在这里需要讲解一下。

在进行Swizzling的时候，我们需要用class_addMethod先进行判断一下原有类中是否有要替换的方法的实现。

如果class_addMethod返回NO，说明当前类中有要替换方法的实现，所以可以直接进行替换，调用method_exchangeImplementations即可实现Swizzling。

如果class_addMethod返回YES，说明当前类中没有要替换方法的实现，我们需要在父类中去寻找。这个时候就需要用到method_getImplementation去获取class_getInstanceMethod里面的方法实现。然后再进行class_replaceMethod来实现Swizzling。

这是Swizzling需要判断的一点。

还有一点需要注意的是，在我们替换的方法- (void)xxx_viewWillAppear:(BOOL)animated中，调用了[self xxx_viewWillAppear:animated];这不是死循环了么？

其实这里并不会死循环。
由于我们进行了Swizzling，所以其实在原来的- (void)viewWillAppear:(BOOL)animated方法中，调用的是- (void)xxx_viewWillAppear:(BOOL)animated方法的实现。所以不会造成死循环。相反的，如果这里把[self xxx_viewWillAppear:animated];改成[self viewWillAppear:animated];就会造成死循环。因为外面调用[self viewWillAppear:animated];的时候，会交换方法走到[self xxx_viewWillAppear:animated];这个方法实现中来，然后这里又去调用[self viewWillAppear:animated]，就会造成死循环了。

所以按照上述Swizzling的模板来写，就不会遇到这4点需要注意的问题啦。

##### (4)Method Swizzling使用场景

a.实现AOP

AOP的例子在上一篇文章中举了一个例子，在下一章中也打算详细分析一下其实现原理，这里就一笔带过。

b.实现埋点统计

如果app有埋点需求，并且要自己实现一套埋点逻辑，那么这里用到Swizzling是很合适的选择。优点在开头已经分析了，这里不再赘述。看到一篇分析的挺精彩的埋点的文章，推荐大家阅读。
[iOS动态性(二)可复用而且高度解耦的用户统计埋点实现](https://www.jianshu.com/p/0497afdad36d)

c.实现异常保护

日常开发我们经常会遇到NSArray数组越界的情况，苹果的API也没有对异常保护，所以需要我们开发者开发时候多多留意。关于Index有好多方法，objectAtIndex，removeObjectAtIndex，replaceObjectAtIndex，exchangeObjectAtIndex等等，这些设计到Index都需要判断是否越界。

常见做法是给NSArray，NSMutableArray增加分类，增加这些异常保护的方法，不过如果原有工程里面已经写了大量的AtIndex系列的方法，去替换成新的分类的方法，效率会比较低。这里可以考虑用Swizzling做。

```
#import "NSArray+ Swizzling.h"
#import "objc/runtime.h"
@implementation NSArray (Swizzling)
+ (void)load {
    Method fromMethod = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(objectAtIndex:));
    Method toMethod = class_getInstanceMethod(objc_getClass("__NSArrayI"), @selector(swizzling_objectAtIndex:));
    method_exchangeImplementations(fromMethod, toMethod);
}

- (id)swizzling_objectAtIndex:(NSUInteger)index {
    if (self.count-1 < index) {
        // 异常处理
        @try {
            return [self swizzling_objectAtIndex:index];
        }
        @catch (NSException *exception) {
            // 打印崩溃信息
            NSLog(@"---------- %s Crash Because Method %s  ----------\n", class_getName(self.class), __func__);
            NSLog(@"%@", [exception callStackSymbols]);
            return nil;
        }
        @finally {}
    } else {
        return [self swizzling_objectAtIndex:index];
    }
}
@end

```

#### 3. Aspect Oriented Programming

类似记录日志、身份验证、缓存等事务非常琐碎，与业务逻辑无关，很多地方都有，又很难抽象出一个模块，这种程序设计问题，业界给它们起了一个名字叫横向关注点(Cross-cutting concern)，AOP作用就是分离横向关注点(Cross-cutting concern)来提高模块复用性，它可以在既有的代码添加一些额外的行为(记录日志、身份验证、缓存)而无需修改代码。

接下来分析分析AOP的工作原理。

在上一篇中我们分析过了，在objc_msgSend函数查找IMP的过程中，如果在父类也没有找到相应的IMP，那么就会开始执行_class_resolveMethod方法，如果不是元类，就执行_class_resolveInstanceMethod，如果是元类，执行_class_resolveClassMethod。在这个方法中，允许开发者动态增加方法实现。这个阶段一般是给@dynamic属性变量提供动态方法的。

如果_class_resolveMethod无法处理，会开始选择备援接受者接受消息，这个时候就到了forwardingTargetForSelector方法。如果该方法返回非nil的对象，则使用该对象作为新的消息接收者。

```
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(Method:)){
        return otherObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

同样也可以替换类方法

```
+ (id)forwardingTargetForSelector:(SEL)aSelector {
    if(aSelector == @selector(xxx)) {
        return NSClassFromString(@"Class name");
    }
    return [super forwardingTargetForSelector:aSelector];
}

```

替换类方法返回值就是一个类对象。

forwardingTargetForSelector这种方法属于单纯的转发，无法对消息的参数和返回值进行处理。

最后到了完整转发阶段。

Runtime系统会向对象发送methodSignatureForSelector:消息，并取到返回的方法签名用于生成NSInvocation对象。为接下来的完整的消息转发生成一个 NSMethodSignature对象。NSMethodSignature 对象会被包装成 NSInvocation 对象，forwardInvocation: 方法里就可以对 NSInvocation 进行处理了。

```
// 为目标对象中被调用的方法返回一个NSMethodSignature实例
#warning 运行时系统要求在执行标准转发时实现这个方法
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel{
    return [self.proxyTarget methodSignatureForSelector:sel];
}
```

对象需要创建一个NSInvocation对象，把消息调用的全部细节封装进去，包括selector, target, arguments 等参数，还能够对返回结果进行处理。

AOP的多数操作就是在forwardInvocation中完成的。一般会分为2个阶段，一个是Intercepter注册阶段，一个是Intercepter执行阶段。

##### (1)Intercepter注册

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9ty0nnlzj31000deq3l.jpg)

首先会把类里面的某个要切片的方法的IMP加入到Aspect中，类方法里面如果有forwardingTargetForSelector:的IMP，也要加入到Aspect中。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9tyscz91j31920eogmt.jpg)

然后对类的切片方法和forwardingTargetForSelector:的IMP进行替换。两者的IMP相应的替换为objc_msgForward()方法和hook过的forwardingTargetForSelector:。这样主要的Intercepter注册就完成了。

##### (2)Intercepter执行

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9tz5d77zj30zs0ckmxr.jpg)

当执行func()方法的时候，会去查找它的IMP，现在它的IMP已经被我们替换为了objc_msgForward()方法，于是开始查找备援转发对象。

查找备援接受者调用forwardingTargetForSelector:这个方法，由于这里是被我们hook过的，所以IMP指向的是hook过的forwardingTargetForSelector:方法。这里我们会返回Aspect的target，即选取Aspect作为备援接受者。

有了备援接受者之后，就会重新objc_msgSend，从消息发送阶段重头开始。

objc_msgSend找不到指定的IMP，再进行_class_resolveMethod，这里也没有找到，forwardingTargetForSelector:这里也不做处理，接着就会methodSignatureForSelector。在methodSignatureForSelector方法中创建一个NSInvocation对象，传递给最终的forwardInvocation方法。

Aspect里面的forwardInvocation方法会干所有切面的事情。这里转发逻辑就完全由我们自定义了。Intercepter注册的时候我们也加入了原来方法中的method()和forwardingTargetForSelector:方法的IMP，这里我们可以在forwardInvocation方法中去执行这些IMP。在执行这些IMP的前后都可以任意的插入任何IMP以达到切面的目的。

Tips:SEL是方法c字符串，IMP是实现方法的地址。

以上就是AOP的原理。

#### 4. Isa Swizzling

前面第二点谈到了黑魔法Method Swizzling，本质上就是对IMP和SEL进行交换。其实接下来要说的Isa Swizzling，和它类似，本质上也是交换，不过交换的是Isa。

在苹果的官方库里面有一个很有名的技术就用到了这个Isa Swizzling，那就是KVO——Key-Value Observing。

KVO是为了监听一个对象的某个属性值是否发生变化。在属性值发生变化的时候，肯定会调用其setter方法。所以KVO的本质就是监听对象有没有调用被监听属性对应的setter方法。具体实现应该是重写其setter方法即可。

官方是如何优雅的实现重写监听类的setter方法的呢？实验代码如下

```
 Student *stu = [[Student alloc]init];
    
    [stu addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
```

我们可以打印观察isa指针的指向

```
Printing description of stu->isa:
Student

Printing description of stu->isa:
NSKVONotifying_Student
```

通过打印，我们可以很明显的看到，被观察的对象的isa变了，变成了NSKVONotifying_Student这个类了。

在@interface NSObject(NSKeyValueObserverRegistration) 这个分类里面，苹果定义了KVO的方法。

```
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;

- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(nullable void *)context NS_AVAILABLE(10_7, 5_0);

- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
```

KVO在调用addObserver方法之后，苹果的做法是在执行完addObserver: forKeyPath: options: context: 方法之后，把isa指向到另外一个类去。

在这个新类里面重写被观察的对象四个方法。

- class
- setter
- dealloc
- _isKVOA

##### （1）重写class方法

重写class方法是为了我们调用它的时候返回跟重写继承类之前同样的内容。

```
static NSArray * ClassMethodNames(Class c)
{
    NSMutableArray * array = [NSMutableArray array];
    unsigned int methodCount = 0;
    Method * methodList = class_copyMethodList(c, &methodCount);
    unsigned int i;
    for(i = 0; i < methodCount; i++) {
        [array addObject: NSStringFromSelector(method_getName(methodList[i]))];
    }
    
    free(methodList);
    return array;
}

int main(int argc, char * argv[]) {
    
    Student *stu = [[Student alloc]init];
    
    NSLog(@"self->isa:%@",object_getClass(stu));
    NSLog(@"self class:%@",[stu class]);
    NSLog(@"ClassMethodNames = %@",ClassMethodNames(object_getClass(stu)));
    [stu addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew context:nil];
    
    NSLog(@"self->isa:%@",object_getClass(stu));
    NSLog(@"self class:%@",[stu class]);
    NSLog(@"ClassMethodNames = %@",ClassMethodNames(object_getClass(stu)));
}

```

打印结果

```
self->isa:Student
self class:Student
ClassMethodNames = (
".cxx_destruct",
name,
"setName:"
)

self->isa:NSKVONotifying_Student
self class:Student
ClassMethodNames = (
"setName:",
class,
dealloc,
"_isKVOA"
)
```

这里也可以看出，这是object_getClass方法和class方法的区别。

这里要特别说明一下，为何打印object_getClass方法和class方法打印出来结果不同。

```
- (Class)class {
    return object_getClass(self);
}

Class object_getClass(id obj)  
{
    if (obj) return obj->getIsa();
    else return Nil;
}
```

从实现上看，两个方法的实现都一样的，按道理来说，打印结果应该相同，可是为何在加了 KVO 以后会出现打印结果不同呢？

根本原因：对于KVO，底层交换了 NSKVONotifying_Student 的 class 方法，让其返回 Student。

打印这句话 object_getClass(stu) 的时候，isa 当然是 NSKVONotifying_Student。

##### (2) 重写setter方法

在新的类中会重写对应的set方法，是为了在set方法中增加另外两个方法的调用：

```
- (void)willChangeValueForKey:(NSString *)key
- (void)didChangeValueForKey:(NSString *)key
```

在didChangeValueForKey:方法再调用

```
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context
```

这里有几种情况需要说明一下：

1)如果使用了KVC
如果有访问器方法，则运行时会在setter方法中调用will/didChangeValueForKey:方法；

如果没用访问器方法，运行时会在setValue:forKey方法中调用will/didChangeValueForKey:方法。

所以这种情况下，KVO是奏效的。

2)有访问器方法
运行时会重写访问器方法调用will/didChangeValueForKey:方法。
因此，直接调用访问器方法改变属性值时，KVO也能监听到。

3)直接调用will/didChangeValueForKey:方法。

综上所述，只要setter中重写will/didChangeValueForKey:方法就可以使用KVO了。

##### (3)重写dealloc方法

销毁新生成的NSKVONotifying_类。

##### (5)重写_isKVOA方法

这个私有方法估计可能是用来标示该类是一个 KVO 机制声称的类。

#### 5. Associated Object关联对象

Associated Objects是Objective-C 2.0中Runtime的特性之一。众所周知，在 Category 中，我们无法添加@property，因为添加了@property之后并不会自动帮我们生成实例变量以及存取方法。那么，我们现在就可以通过关联对象来实现在 Category 中添加属性的功能了。

##### (1)用法

```
// NSObject+AssociatedObject.h
@interface NSObject (AssociatedObject)
@property (nonatomic, strong) id associatedObject;
@end

// NSObject+AssociatedObject.m
@implementation NSObject (AssociatedObject)
@dynamic associatedObject;

- (void)setAssociatedObject:(id)object {
    objc_setAssociatedObject(self, @selector(associatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)associatedObject {
    return objc_getAssociatedObject(self, @selector(associatedObject));
}
```

这里涉及到了3个函数：

```
OBJC_EXPORT void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);

OBJC_EXPORT id objc_getAssociatedObject(id object, const void *key)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);

OBJC_EXPORT void objc_removeAssociatedObjects(id object)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);
```

来说明一下这些参数的意义：

1.id object 设置关联对象的实例对象

2.const void *key 区分不同的关联对象的 key。这里会有3种写法。

使用 &AssociatedObjectKey 作为key值

`static char AssociatedObjectKey = "AssociatedKey";`

使用AssociatedKey 作为key值

`static const void *AssociatedKey = "AssociatedKey";`

使用@selector

`@selector(associatedKey)`

3种方法都可以，不过推荐使用更加简洁的第三种方式。

3.id value 关联的对象

4.objc_AssociationPolicy policy 关联对象的存储策略，它是一个枚举，与property的attribute 相对应。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9ulb1qanj30zq0pswhj.jpg)

这里需要注意的是标记成OBJC_ASSOCIATION_ASSIGN的关联对象和
@property (weak) 是不一样的，上面表格中等价定义写的是 @property (unsafe_unretained)，对象被销毁时，属性值仍然还在。如果之后再次使用该对象就会导致程序闪退。所以我们在使用OBJC_ASSOCIATION_ASSIGN时，要格外注意。

关于关联对象还有一点需要说明的是objc_removeAssociatedObjects。这个方法是移除源对象中所有的关联对象，并不是其中之一。所以其方法参数中也没有传入指定的key。要删除指定的关联对象，使用 objc_setAssociatedObject 方法将对应的 key 设置成 nil 即可。

```
objc_setAssociatedObject(self, associatedKey, nil, OBJC_ASSOCIATION_COPY_NONATOMIC);
```

关联对象3种使用场景

1.为现有的类添加私有变量
2.为现有的类添加公有属性
3.为KVO创建一个关联的观察者。

##### (2)源码分析

1) objc_setAssociatedObject方法

```
void _object_set_associative_reference(id object, void *key, id value, uintptr_t policy) {
    // retain the new value (if any) outside the lock.
    ObjcAssociation old_association(0, nil);
    id new_value = value ? acquireValue(value, policy) : nil;
    {
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.associations());
        disguised_ptr_t disguised_object = DISGUISE(object);
        if (new_value) {
            // break any existing association.
            AssociationsHashMap::iterator i = associations.find(disguised_object);
            if (i != associations.end()) {
                // secondary table exists
                ObjectAssociationMap *refs = i->second;
                ObjectAssociationMap::iterator j = refs->find(key);
                if (j != refs->end()) {
                    old_association = j->second;
                    j->second = ObjcAssociation(policy, new_value);
                } else {
                    (*refs)[key] = ObjcAssociation(policy, new_value);
                }
            } else {
                // create the new association (first time).
                ObjectAssociationMap *refs = new ObjectAssociationMap;
                associations[disguised_object] = refs;
                (*refs)[key] = ObjcAssociation(policy, new_value);
                object->setHasAssociatedObjects();
            }
        } else {
            // setting the association to nil breaks the association.
            AssociationsHashMap::iterator i = associations.find(disguised_object);
            if (i !=  associations.end()) {
                ObjectAssociationMap *refs = i->second;
                ObjectAssociationMap::iterator j = refs->find(key);
                if (j != refs->end()) {
                    old_association = j->second;
                    refs->erase(j);
                }
            }
        }
    }
    // release the old value (outside of the lock).
    if (old_association.hasValue()) ReleaseValue()(old_association);
}
```

这个函数里面主要分为2部分，一部分是if里面对应的new_value不为nil的时候，另一部分是else里面对应的new_value为nil的情况。

当new_value不为nil的时候，查找时候，流程如下：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9uu5vlxzj31gi0rk40p.jpg)

首先在AssociationsManager的结构如下

```
class AssociationsManager {
    static spinlock_t _lock;
    static AssociationsHashMap *_map;
public:
    AssociationsManager()   { _lock.lock(); }
    ~AssociationsManager()  { _lock.unlock(); }
    
    AssociationsHashMap &associations() {
        if (_map == NULL)
            _map = new AssociationsHashMap();
        return *_map;
    }
};
```

在AssociationsManager中有一个spinlock类型的自旋锁lock。保证每次只有一个线程对AssociationsManager进行操作，保证线程安全。AssociationsHashMap对应的是一张哈希表。

AssociationsHashMap哈希表里面key是disguised_ptr_t。

`disguised_ptr_t disguised_object = DISGUISE(object);`

通过调用DISGUISE( )方法获取object地址的指针。拿到disguised_object后，通过这个key值，在AssociationsHashMap哈希表里面找到对应的value值。而这个value值ObjcAssociationMap表的首地址。

在ObjcAssociationMap表中，key值是set方法里面传过来的形参const void *key，value值是ObjcAssociation对象。

ObjcAssociation对象中存储了set方法最后两个参数，policy和value。

所以objc_setAssociatedObject方法中传的4个形参在上图中已经标出。

现在弄清楚结构之后再来看源码，就很容易了。objc_setAssociatedObject方法的目的就是在这2张哈希表中存储对应的键值对。

先初始化一个 AssociationsManager，获取唯一的保存关联对象的哈希表 AssociationsHashMap，然后在AssociationsHashMap里面去查找object地址的指针。

如果找到，就找到了第二张表ObjectAssociationMap。在这张表里继续查找object的key。

```
if (i != associations.end()) {
    // secondary table exists
    ObjectAssociationMap *refs = i->second;
    ObjectAssociationMap::iterator j = refs->find(key);
    if (j != refs->end()) {
        old_association = j->second;
        j->second = ObjcAssociation(policy, new_value);
    } else {
        (*refs)[key] = ObjcAssociation(policy, new_value);
    }
}
```

如果在第二张表ObjectAssociationMap找到对应的ObjcAssociation对象，那就更新它的值。如果没有找到，就新建一个ObjcAssociation对象，放入第二张表ObjectAssociationMap中。

再回到第一张表AssociationsHashMap中，如果没有找到对应的键值

```
ObjectAssociationMap *refs = new ObjectAssociationMap;
associations[disguised_object] = refs;
(*refs)[key] = ObjcAssociation(policy, new_value);
object->setHasAssociatedObjects();
```

此时就不存在第二张表ObjectAssociationMap了，这时就需要新建第二张ObjectAssociationMap表，来维护对象的所有新增属性。新建完第二张ObjectAssociationMap表之后，还需要再实例化 ObjcAssociation对象添加到 Map 中，调用setHasAssociatedObjects方法，表明当前对象含有关联对象。这里的setHasAssociatedObjects方法，改变的是isa_t结构体中的第二个标志位has_assoc的值。(关于isa_t结构体的结构，详情请看第一天的解析)

```
// release the old value (outside of the lock).
 if (old_association.hasValue()) ReleaseValue()(old_association);
```

最后如果老的association对象有值，此时还会释放它。

以上是new_value不为nil的情况。其实只要记住上面那2张表的结构，这个objc_setAssociatedObject的过程就是更新 / 新建 表中键值对的过程。

再来看看new_value为nil的情况

```
// setting the association to nil breaks the association.
AssociationsHashMap::iterator i = associations.find(disguised_object);
if (i !=  associations.end()) {
    ObjectAssociationMap *refs = i->second;
    ObjectAssociationMap::iterator j = refs->find(key);
    if (j != refs->end()) {
        old_association = j->second;
        refs->erase(j);
    }
}
```

当new_value为nil的时候，就是我们要移除关联对象的时候。这个时候就是在两张表中找到对应的键值，并调用erase( )方法，即可删除对应的关联对象。

2) objc_getAssociatedObject方法

```
id _object_get_associative_reference(id object, void *key) {
    id value = nil;
    uintptr_t policy = OBJC_ASSOCIATION_ASSIGN;
    {
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.associations());
        disguised_ptr_t disguised_object = DISGUISE(object);
        AssociationsHashMap::iterator i = associations.find(disguised_object);
        if (i != associations.end()) {
            ObjectAssociationMap *refs = i->second;
            ObjectAssociationMap::iterator j = refs->find(key);
            if (j != refs->end()) {
                ObjcAssociation &entry = j->second;
                value = entry.value();
                policy = entry.policy();
                if (policy & OBJC_ASSOCIATION_GETTER_RETAIN) ((id(*)(id, SEL))objc_msgSend)(value, SEL_retain);
            }
        }
    }
    if (value && (policy & OBJC_ASSOCIATION_GETTER_AUTORELEASE)) {
        ((id(*)(id, SEL))objc_msgSend)(value, SEL_autorelease);
    }
    return value;
}
```

objc_getAssociatedObject方法 很简单。就是通过遍历AssociationsHashMap哈希表 和 ObjcAssociationMap表的所有键值找到对应的ObjcAssociation对象，找到了就返回ObjcAssociation对象，没有找到就返回nil。

3) objc_removeAssociatedObjects方法

```
void objc_removeAssociatedObjects(id object) {
    if (object && object->hasAssociatedObjects()) {
        _object_remove_assocations(object);
    }
}

void _object_remove_assocations(id object) {
    vector< ObjcAssociation,ObjcAllocator<ObjcAssociation> > elements;
    {
        AssociationsManager manager;
        AssociationsHashMap &associations(manager.associations());
        if (associations.size() == 0) return;
        disguised_ptr_t disguised_object = DISGUISE(object);
        AssociationsHashMap::iterator i = associations.find(disguised_object);
        if (i != associations.end()) {
            // copy all of the associations that need to be removed.
            ObjectAssociationMap *refs = i->second;
            for (ObjectAssociationMap::iterator j = refs->begin(), end = refs->end(); j != end; ++j) {
                elements.push_back(j->second);
            }
            // remove the secondary table.
            delete refs;
            associations.erase(i);
        }
    }
    // the calls to releaseValue() happen outside of the lock.
    for_each(elements.begin(), elements.end(), ReleaseValue());
}
```

在移除关联对象object的时候，会先去判断object的isa_t中的第二位has_assoc的值，当object 存在并且object->hasAssociatedObjects( )值为1的时候，才会去调用_object_remove_assocations方法。

_object_remove_assocations方法的目的是删除第二张ObjcAssociationMap表，即删除所有的关联对象。删除第二张表，就需要在第一张AssociationsHashMap表中遍历查找。这里会把第二张ObjcAssociationMap表中所有的ObjcAssociation对象都存到一个数组elements里面，然后调用associations.erase( )删除第二张表。最后再遍历elements数组，把ObjcAssociation对象依次释放。

以上就是Associated Object关联对象3个函数的源码分析。

#### 6. 动态的增加方法

在消息发送阶段，如果在父类中也没有找到相应的IMP，就会执行resolveInstanceMethod方法。在这个方法里面，我们可以动态的给类对象或者实例对象动态的增加方法。

```
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    
    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString isEqualToString:@"method1"]) {
        class_addMethod(self.class, @selector(method1), (IMP)functionForMethod1, "@:");
    }
    
    return [super resolveInstanceMethod:sel];
}
```

关于方法操作方面的函数还有以下这些

```
// 调用指定方法的实现
id method_invoke ( id receiver, Method m, ... );
// 调用返回一个数据结构的方法的实现
void method_invoke_stret ( id receiver, Method m, ... );
// 获取方法名
SEL method_getName ( Method m );
// 返回方法的实现
IMP method_getImplementation ( Method m );
// 获取描述方法参数和返回值类型的字符串
const char * method_getTypeEncoding ( Method m );
// 获取方法的返回值类型的字符串
char * method_copyReturnType ( Method m );
// 获取方法的指定位置参数的类型字符串
char * method_copyArgumentType ( Method m, unsigned int index );
// 通过引用返回方法的返回值类型字符串
void method_getReturnType ( Method m, char *dst, size_t dst_len );
// 返回方法的参数的个数
unsigned int method_getNumberOfArguments ( Method m );
// 通过引用返回方法指定位置参数的类型字符串
void method_getArgumentType ( Method m, unsigned int index, char *dst, size_t dst_len );
// 返回指定方法的方法描述结构体
struct objc_method_description * method_getDescription ( Method m );
// 设置方法的实现
IMP method_setImplementation ( Method m, IMP imp );
// 交换两个方法的实现
void method_exchangeImplementations ( Method m1, Method m2 );
```

这些方法其实平时不需要死记硬背，使用的时候只要先打出method开头，后面就会有补全信息，找到相应的方法，传入对应的方法即可。

#### 7. NSCoding的自动归档和自动解档

现在虽然手写归档和解档的时候不多了，但是自动操作还是用Runtime来实现的。

```
- (void)encodeWithCoder:(NSCoder *)aCoder{
    [aCoder encodeObject:self.name forKey:@"name"];
}

- (id)initWithCoder:(NSCoder *)aDecoder{
    if (self = [super init]) {
        self.name = [aDecoder decodeObjectForKey:@"name"];
    }
    return self;
}

```

手动的有一个缺陷，如果属性多起来，要写好多行相似的代码，虽然功能是可以完美实现，但是看上去不是很优雅。

用runtime实现的思路就比较简单，我们循环依次找到每个成员变量的名称，然后利用KVC读取和赋值就可以完成encodeWithCoder和initWithCoder了。

```
#import "Student.h"
#import <objc/runtime.h>
#import <objc/message.h>

@implementation Student

- (void)encodeWithCoder:(NSCoder *)aCoder{
    unsigned int outCount = 0;
    Ivar *vars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar var = vars[i];
        const char *name = ivar_getName(var);
        NSString *key = [NSString stringWithUTF8String:name];

        id value = [self valueForKey:key];
        [aCoder encodeObject:value forKey:key];
    }
}

- (nullable __kindof)initWithCoder:(NSCoder *)aDecoder{
    if (self = [super init]) {
        unsigned int outCount = 0;
        Ivar *vars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar var = vars[i];
            const char *name = ivar_getName(var);
            NSString *key = [NSString stringWithUTF8String:name];
            id value = [aDecoder decodeObjectForKey:key];
            [self setValue:value forKey:key];
        }
    }
    return self;
}
@end

```

class_copyIvarList方法用来获取当前 Model 的所有成员变量，ivar_getName方法用来获取每个成员变量的名称。

#### 8. 字典和模型互相转换

##### (1) 字典(Dic)转模型(Model)

1.调用 class_getProperty 方法获取当前 Model 的所有属性。
2.调用 property_copyAttributeList 获取属性列表。
3.根据属性名称生成 setter 方法。
4.使用 objc_msgSend 调用 setter 方法为 Model 的属性赋值（或者 KVC）

```
+(id)objectWithKeyValues:(NSDictionary *)aDictionary{
    id objc = [[self alloc] init];
    for (NSString *key in aDictionary.allKeys) {
        id value = aDictionary[key];
        
        /*判断当前属性是不是Model*/
        objc_property_t property = class_getProperty(self, key.UTF8String);
        unsigned int outCount = 0;
        objc_property_attribute_t *attributeList = property_copyAttributeList(property, &outCount);
        objc_property_attribute_t attribute = attributeList[0];
        NSString *typeString = [NSString stringWithUTF8String:attribute.value];

        if ([typeString isEqualToString:@"@\"Student\""]) {
            value = [self objectWithKeyValues:value];
        }
        
        //生成setter方法，并用objc_msgSend调用
        NSString *methodName = [NSString stringWithFormat:@"set%@%@:",[key substringToIndex:1].uppercaseString,[key substringFromIndex:1]];
        SEL setter = sel_registerName(methodName.UTF8String);
        if ([objc respondsToSelector:setter]) {
            ((void (*) (id,SEL,id)) objc_msgSend) (objc,setter,value);
        }
        free(attributeList);
    }
    return objc;
}
```

这段代码里面有一处判断typeString的，这里判断是防止model嵌套，比如说Student里面还有一层Student，那么这里就需要再次转换一次，当然这里有几层就需要转换几次。

几个出名的开源库JSONModel、MJExtension等都是通过这种方式实现的(利用runtime的class_copyIvarList获取属性数组，遍历模型对象的所有成员属性，根据属性名找到字典中key值进行赋值，当然这种方法只能解决NSString、NSNumber等，如果含有NSArray或NSDictionary，还要进行第二步转换，如果是字典数组，需要遍历数组中的字典，利用objectWithDict方法将字典转化为模型，在将模型放到数组中，最后把这个模型数组赋值给之前的字典数组)

##### (2)模型(Model)转字典(Dic)

这里是上一部分字典转模型的逆步骤：

1.调用 class_copyPropertyList 方法获取当前 Model 的所有属性。
2.调用 property_getName 获取属性名称。
3.根据属性名称生成 getter 方法。
4.使用 objc_msgSend 调用 getter 方法获取属性值（或者 KVC）

```
//模型转字典
-(NSDictionary *)keyValuesWithObject{
    unsigned int outCount = 0;
    objc_property_t *propertyList = class_copyPropertyList([self class], &outCount);
    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
    for (int i = 0; i < outCount; i ++) {
        objc_property_t property = propertyList[i];
        
        //生成getter方法，并用objc_msgSend调用
        const char *propertyName = property_getName(property);
        SEL getter = sel_registerName(propertyName);
        if ([self respondsToSelector:getter]) {
            id value = ((id (*) (id,SEL)) objc_msgSend) (self,getter);
            
            /*判断当前属性是不是Model*/
            if ([value isKindOfClass:[self class]] && value) {
                value = [value keyValuesWithObject];
            }

            if (value) {
                NSString *key = [NSString stringWithUTF8String:propertyName];
                [dict setObject:value forKey:key];
            }
        }
        
    }
    free(propertyList);
    return dict;
}
```

中间注释那里的判断也是防止model嵌套，如果model里面还有一层model，那么model转字典的时候还需要再次转换，同样，有几层就需要转换几次。

不过上述的做法是假设字典里面不再包含二级字典，如果还包含数组，数组里面再包含字典，那还需要多级转换。这里有一个关于字典里面包含数组的demo.

### （二）Runtime 的缺点

#### 1. Method swizzling is not atomic

Method swizzling不是原子性操作。如果在+load方法里面写，是没有问题的，但是如果写在+initialize方法中就会出现一些奇怪的问题。

#### 2. Changes behavior of un-owned code

如果你在一个类中重写一个方法，并且不调用super方法，你可能会导致一些问题出现。在大多数情况下，super方法是期望被调用的（除非有特殊说明）。如果你使用同样的思想来进行Swizzling，可能就会引起很多问题。如果你不调用原始的方法实现，那么你Swizzling改变的太多了，而导致整个程序变得不安全。



#### 3. Possible naming conflicts

命名冲突是程序开发中经常遇到的一个问题。我们经常在类别中的前缀类名称和方法名称。不幸的是，命名冲突是在我们程序中的像一种瘟疫。一般我们会这样写Method Swizzling

```
@interface NSView : NSObject
- (void)setFrame:(NSRect)frame;
@end

@implementation NSView (MyViewAdditions)

- (void)my_setFrame:(NSRect)frame {
    // do custom work
    [self my_setFrame:frame];
}

+ (void)load {
    [self swizzle:@selector(setFrame:) with:@selector(my_setFrame:)];
}

@end
```

这样写看上去是没有问题的。但是如果在整个大型程序中还有另外一处定义了my_setFrame:方法呢？那又会造成命名冲突的问题。我们应该把上面的Swizzling改成以下这种样子：

```
@implementation NSView (MyViewAdditions)

static void MySetFrame(id self, SEL _cmd, NSRect frame);
static void (*SetFrameIMP)(id self, SEL _cmd, NSRect frame);

static void MySetFrame(id self, SEL _cmd, NSRect frame) {
    // do custom work
    SetFrameIMP(self, _cmd, frame);
}

+ (void)load {
    [self swizzle:@selector(setFrame:) with:(IMP)MySetFrame store:(IMP *)&SetFrameIMP];
}

@end
```

虽然上面的代码看上去不是OC(因为使用了函数指针)，但是这种做法确实有效的防止了命名冲突的问题。原则上来说，其实上述做法更加符合标准化的Swizzling。这种做法可能和人们使用方法不同，但是这种做法更好。Swizzling Method 标准定义应该是如下的样子：

```
typedef IMP *IMPPointer;

BOOL class_swizzleMethodAndStore(Class class, SEL original, IMP replacement, IMPPointer store) {
    IMP imp = NULL;
    Method method = class_getInstanceMethod(class, original);
    if (method) {
        const char *type = method_getTypeEncoding(method);
        imp = class_replaceMethod(class, original, replacement, type);
        if (!imp) {
            imp = method_getImplementation(method);
        }
    }
    if (imp && store) { *store = imp; }
    return (imp != NULL);
}

@implementation NSObject (FRRuntimeAdditions)
+ (BOOL)swizzle:(SEL)original with:(IMP)replacement store:(IMPPointer)store {
    return class_swizzleMethodAndStore(self, original, replacement, store);
}
@end
```

#### 4. Swizzling changes the method's arguments

这一点是这些问题中最大的一个。标准的Method Swizzling是不会改变方法参数的。使用Swizzling中，会改变传递给原来的一个函数实现的参数，例如：

`[self my_setFrame:frame];`

会变转换成

`objc_msgSend(self, @selector(my_setFrame:), frame);`

objc_msgSend会去查找my_setFrame对应的IMP。一旦IMP找到，会把相同的参数传递进去。这里会找到最原始的setFrame:方法，调用执行它。但是这里的_cmd参数并不是setFrame:，现在是my_setFrame:。原始的方法就被一个它不期待的接收参数调用了。这样并不好。

这里有一个简单的解决办法，上一条里面所说的，用函数指针去实现。参数就不会变了。

#### 5. The order of swizzles matters

调用顺序对于Swizzling来说，很重要。假设setFrame:方法仅仅被定义在NSView类里面。

```
[NSButton swizzle:@selector(setFrame:) with:@selector(my_buttonSetFrame:)];
[NSControl swizzle:@selector(setFrame:) with:@selector(my_controlSetFrame:)];
[NSView swizzle:@selector(setFrame:) with:@selector(my_viewSetFrame:)];
```

当NSButton被swizzled之后会发生什么呢？大多数的swizzling应该保证不会替换setFrame:方法。因为一旦改了这个方法，会影响下面所有的View。所以它会去拉取实例方法。NSButton会使用已经存在的方法去重新定义setFrame:方法。以至于改变了IMP实现不会影响所有的View。相同的事情也会发生在对NSControl进行swizzling的时候，同样，IMP也是定义在NSView类里面，把NSControl 和 NSButton这上下两行swizzle顺序替换，结果也是相同的。

当调用NSButton的setFrame:方法，会去调用swizzled method，然后会跳入NSView类里面定义的setFrame:方法。NSControl 和 NSView对应的swizzled method不会被调用。

NSButton 和 NSControl各自调用各自的 swizzling方法，相互不会影响。

但是我们改变一下调用顺序，把NSView放在第一位调用。

```
[NSView swizzle:@selector(setFrame:) with:@selector(my_viewSetFrame:)];
[NSControl swizzle:@selector(setFrame:) with:@selector(my_controlSetFrame:)];
[NSButton swizzle:@selector(setFrame:) with:@selector(my_buttonSetFrame:)];
```

一旦这里的NSView先进行了swizzling了以后，情况就和上面大不相同了。NSControl的swizzling会去拉取NSView替换后的方法。相应的，NSControl在NSButton前面，NSButton也会去拉取到NSControl替换后的方法。这样就十分混乱了。但是顺序就是这样排列的。我们开发中如何能保证不出现这种混乱呢？

再者，在load方法中加载swizzle。如果仅仅是在已经加载完成的class中做了swizzle，那么这样做是安全的。load方法能保证父类会在其任何子类加载方法之前，加载相应的方法。这就保证了我们调用顺序的正确性。

#### 6. Difficult to understand (looks recursive)

看着传统定义的swizzled method，我认为很难去预测会发生什么。但是对比上面标准的swizzling，还是很容易明白。这一点已经被解决了。

#### 7. Difficult to debug

在调试中，会出现奇怪的堆栈调用信息，尤其是swizzled的命名很混乱，一切方法调用都是混乱的。对比标准的swizzled方式，你会在堆栈中看到清晰的命名方法。swizzling还有一个比较难调试的一点， 在于你很难记住当前确切的哪个方法已经被swizzling了。

在代码里面写好文档注释，即使你认为这段代码只有你一个人会看。遵循这个方式去实践，你的代码都会没问题。它的调试也没有多线程的调试困难。


在合理使用 + 文档完整齐全 的情况下，解决特定问题，使用Runtime还是非常简洁安全的。

日常可能用的比较多的Runtime函数可能就是下面这些

```
//获取cls类对象所有成员ivar结构体
Ivar *class_copyIvarList(Class cls, unsigned int *outCount)
//获取cls类对象name对应的实例方法结构体
Method class_getInstanceMethod(Class cls, SEL name)
//获取cls类对象name对应类方法结构体
Method class_getClassMethod(Class cls, SEL name)
//获取cls类对象name对应方法imp实现
IMP class_getMethodImplementation(Class cls, SEL name)
//测试cls对应的实例是否响应sel对应的方法
BOOL class_respondsToSelector(Class cls, SEL sel)
//获取cls对应方法列表
Method *class_copyMethodList(Class cls, unsigned int *outCount)
//测试cls是否遵守protocol协议
BOOL class_conformsToProtocol(Class cls, Protocol *protocol)
//为cls类对象添加新方法
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)
//替换cls类对象中name对应方法的实现
IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types)
//为cls添加新成员
BOOL class_addIvar(Class cls, const char *name, size_t size, uint8_t alignment, const char *types)
//为cls添加新属性
BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)
//获取m对应的选择器
SEL method_getName(Method m)
//获取m对应的方法实现的imp指针
IMP method_getImplementation(Method m)
//获取m方法的对应编码
const char *method_getTypeEncoding(Method m)
//获取m方法参数的个数
unsigned int method_getNumberOfArguments(Method m)
//copy方法返回值类型
char *method_copyReturnType(Method m)
//获取m方法index索引参数的类型
char *method_copyArgumentType(Method m, unsigned int index)
//获取m方法返回值类型
void method_getReturnType(Method m, char *dst, size_t dst_len)
//获取方法的参数类型
void method_getArgumentType(Method m, unsigned int index, char *dst, size_t dst_len)
//设置m方法的具体实现指针
IMP method_setImplementation(Method m, IMP imp)
//交换m1，m2方法对应具体实现的函数指针
void method_exchangeImplementations(Method m1, Method m2)
//获取v的名称
const char *ivar_getName(Ivar v)
//获取v的类型编码
const char *ivar_getTypeEncoding(Ivar v)
//设置object对象关联的对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
//获取object关联的对象
id objc_getAssociatedObject(id object, const void *key)
//移除object关联的对象
void objc_removeAssociatedObjects(id object)
```

这些API看上去不好记，其实使用的时候不难，关于方法操作的，一般都是method开头，关于类的，一般都是class开头的，其他的基本都是objc开头的，剩下的就看代码补全的提示，看方法名基本就能找到想要的方法了。当然很熟悉的话，可以直接打出指定方法，也不会依赖代码补全。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)