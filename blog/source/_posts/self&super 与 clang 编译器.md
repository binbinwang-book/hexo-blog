---
title: self&super 与 clang 编译器
date: 2021-08-09 01:01
tags: [clang]
categories: [iOS]
---

# self&super 与 clang 编译器

阅读完本文你可以收获：

- 明确：self和super的区别
- 学会clang的用法
- 了解 TaggedPoint

source：[](https://juejin.cn/post/6844903896276533255)
source：[个人对于super的调用过程中，一些不一样的理解](https://segmentfault.com/a/1190000009346559)

## 一、self 和 super

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt94g8bwyvj30k40bogmd.jpg)

结果都是：Phone。

这个题目主要是考察关于 Objective-C 中对 self 和 super 的理解。

我们都知道：self 是类的隐藏参数，指向当前调用方法的这个类的实例。那 super 呢？

很多人会想当然的认为“ super 和 self 类似，应该是指向父类的指针吧！”。这是很普遍的一个误区。其实 super 是一个 Magic Keyword， 它本质是一个编译器标示符，和 self 是指向的同一个消息接受者！他们两个的不同点在于：super 会告诉编译器，调用 class 这个方法时，要去父类的方法，而不是本类里的。

上面的例子不管调用[self class]还是[super class]，接受消息的对象都是当前Son ＊xxx这个对象。

当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用 super 时，则从父类的方法列表中开始找。然后调用父类的这个方法。

这也就是为什么说“不推荐在 init 方法中使用点语法”，如果想访问实例变量 iVar 应该使用下划线（_iVar ），而非点语法（ self.iVar ）。

## 二、为什么不建议在init方法中使用点语法

点语法（self.iVar）的坏处就是子类有可能覆写 setter 。

我们看一下下面这个结构的UML图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt94wwgjfmj306305r3yk.jpg)

关键代码为：

```
BNTestClass.h

@interface BNTestClass : NSObject

@property (nonatomic, copy) NSString *lastName;

@end
```

```
BNTestClass.m

@implementation BNTestClass

- (instancetype)init {
    if (self = [super init]) {
        self.lastName = @"";
    }
    return self;
}
@end
```

```
BNTestSubclass.h

@interface BNTestSubclass : BNTestClass

@end
```

```
BNTestSubclass.m

@implementation BNTestSubclass
@synthesize lastName = _lastName;

- (void)setLastName:(NSString *)lastName {
    _lastName = @"BNineCoding";
}

@end
```

`BNTestClass` 在 init 方法中将 `lastName` 设置为 @""，但因为采用的点语法进行设置且子类覆写了`- (void)setLastName:(NSString *)lastName`方法，就会导致在`[[BNTestSubclass alloc] init]`时，直接将 `lastName` 重置为了 `BNineCoding`，而我们的本意是初始化时应该为空字符串。

调用日志截图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt951v96mpj30t109g752.jpg)

这里我们发现，使用 synthesize 的场景不仅是property所在类的 readonly/readwrite 规则被破坏，当子类想覆写父类property 的 set 方法时，也需要使用 synthesize。

## 三、通过 clang 弄清楚 self 和 super 的本质

Clang是一个C、C++、OC语言的轻量级编译器。

Clang由Apple公司开发，源代码授权使用BSD的开源授权。

Clang是由C++编写，基于LLVM，发布于LLVM BSD许可证下的编译器。

操作步骤：

- 0. 将 terminal 切到项目工程下
- 1. 执行 clang -rewrite-objc xxxx.m
- 2. 执行完毕后，会看到项目目录下会生成 xxxx.cpp 文件，cpp作为runtime的源码，对学习runtime非常有用。

我们对 `BNTestSubclass` 进行编译:

```
BNTestSubclass.h 

@interface BNTestSubclass : BNTestClass

@end
```

```
BNTestSubclass.m 

@implementation BNTestSubclass

- (instancetype)init {
    if (self = [super init]) {
        NSLog(@"%@",NSStringFromClass([self class]));
        NSLog(@"%@",NSStringFromClass([super class]));
    }
    return self;
}

@end
```

编译出的 cpp 文件部分内容是：

```
#ifndef __OBJC2__
#define __OBJC2__
#endif
struct objc_selector; struct objc_class;
struct __rw_objc_super { 
	struct objc_object *object; 
	struct objc_object *superClass; 
	__rw_objc_super(struct objc_object *o, struct objc_object *s) : object(o), superClass(s) {} 
};
#ifndef _REWRITER_typedef_Protocol
typedef struct objc_object Protocol;
#define _REWRITER_typedef_Protocol
#endif
#define __OBJC_RW_DLLIMPORT extern
__OBJC_RW_DLLIMPORT void objc_msgSend(void);
__OBJC_RW_DLLIMPORT void objc_msgSendSuper(void);
__OBJC_RW_DLLIMPORT void objc_msgSend_stret(void);
__OBJC_RW_DLLIMPORT void objc_msgSendSuper_stret(void);
__OBJC_RW_DLLIMPORT void objc_msgSend_fpret(void);
__OBJC_RW_DLLIMPORT struct objc_class *objc_getClass(const char *);
__OBJC_RW_DLLIMPORT struct objc_class *class_getSuperclass(struct objc_class *);
__OBJC_RW_DLLIMPORT struct objc_class *objc_getMetaClass(const char *);
__OBJC_RW_DLLIMPORT void objc_exception_throw( struct objc_object *);
__OBJC_RW_DLLIMPORT int objc_sync_enter( struct objc_object *);
__OBJC_RW_DLLIMPORT int objc_sync_exit( struct objc_object *);
__OBJC_RW_DLLIMPORT Protocol *objc_getProtocol(const char *);
#ifdef _WIN64
typedef unsigned long long  _WIN_NSUInteger;
#else
typedef unsigned int _WIN_NSUInteger;
#endif
#ifndef __FASTENUMERATIONSTATE
struct __objcFastEnumerationState {
	unsigned long state;
	void **itemsPtr;
	unsigned long *mutationsPtr;
	unsigned long extra[5];
};
__OBJC_RW_DLLIMPORT void objc_enumerationMutation(struct objc_object *);
#define __FASTENUMERATIONSTATE
#endif
#ifndef __NSCONSTANTSTRINGIMPL
struct __NSConstantStringImpl {
  int *isa;
  int flags;
  char *str;
#if _WIN64
  long long length;
#else
  long length;
#endif
};
#ifdef CF_EXPORT_CONSTANT_STRING
extern "C" __declspec(dllexport) int __CFConstantStringClassReference[];
#else
__OBJC_RW_DLLIMPORT int __CFConstantStringClassReference[];
#endif
#define __NSCONSTANTSTRINGIMPL
#endif
#ifndef BLOCK_IMPL
#define BLOCK_IMPL
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};
// Runtime copy/destroy helper functions (from Block_private.h)
#ifdef __OBJC_EXPORT_BLOCKS
extern "C" __declspec(dllexport) void _Block_object_assign(void *, const void *, const int);
extern "C" __declspec(dllexport) void _Block_object_dispose(const void *, const int);
extern "C" __declspec(dllexport) void *_NSConcreteGlobalBlock[32];
extern "C" __declspec(dllexport) void *_NSConcreteStackBlock[32];
#else
__OBJC_RW_DLLIMPORT void _Block_object_assign(void *, const void *, const int);
__OBJC_RW_DLLIMPORT void _Block_object_dispose(const void *, const int);
__OBJC_RW_DLLIMPORT void *_NSConcreteGlobalBlock[32];
__OBJC_RW_DLLIMPORT void *_NSConcreteStackBlock[32];
#endif
#endif
#define __block
#define __weak

#include <stdarg.h>
struct __NSContainer_literal {
  void * *arr;
  __NSContainer_literal (unsigned int count, ...) {
	va_list marker;
	va_start(marker, count);
	arr = new void *[count];
	for (unsigned i = 0; i < count; i++)
	  arr[i] = va_arg(marker, void *);
	va_end( marker );
  };
  ~__NSContainer_literal() {
	delete[] arr;
  }
};
extern "C" __declspec(dllimport) void * objc_autoreleasePoolPush(void);
extern "C" __declspec(dllimport) void objc_autoreleasePoolPop(void *);

struct __AtAutoreleasePool {
  __AtAutoreleasePool() {atautoreleasepoolobj = objc_autoreleasePoolPush();}
  ~__AtAutoreleasePool() {objc_autoreleasePoolPop(atautoreleasepoolobj);}
  void * atautoreleasepoolobj;
};

#define __OFFSETOFIVAR__(TYPE, MEMBER) ((long long) &((TYPE *)0)->MEMBER)
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_9b_w0ymsg0n3yqdlb90w49xqmz40000gn_T_BNTestSubclass_711894_mi_0 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"%@",2};
static __NSConstantStringImpl __NSConstantStringImpl__var_folders_9b_w0ymsg0n3yqdlb90w49xqmz40000gn_T_BNTestSubclass_711894_mi_1 __attribute__ ((section ("__DATA, __cfstring"))) = {__CFConstantStringClassReference,0x000007c8,"%@",2};
```

我们着重看一下下面这个编译的内容:

```
static instancetype _I_BNTestSubclass_init(BNTestSubclass * self, SEL _cmd) {
    if (self = ((BNTestSubclass *(*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("BNTestSubclass"))}, sel_registerName("init"))) {
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_9b_w0ymsg0n3yqdlb90w49xqmz40000gn_T_BNTestSubclass_711894_mi_0,NSStringFromClass(((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class"))));
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_9b_w0ymsg0n3yqdlb90w49xqmz40000gn_T_BNTestSubclass_711894_mi_1,NSStringFromClass(((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("BNTestSubclass"))}, sel_registerName("class"))));
    }
    return self;
}
```

从上面的代码中，我们可以发现在调用 [self class] 时，会转化成objc_msgSend函数。看下函数定义：

```
 id objc_msgSend(id self, SEL op, ...)
```

我们把 self 做为第一个参数传递进去。

而在调用 [super class]时，会转化成 objc_msgSendSuper函数。看下函数定义:

```
 id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
```

第一个参数是 objc_super这样一个结构体，其定义如下:

```
struct objc_super {
       __unsafe_unretained id receiver;
       __unsafe_unretained Class super_class;
};
```

结构体有两个成员，第一个成员是 receiver, 类似于上面的 objc_msgSend函数第一个参数self 。第二个成员是记录当前类的父类是什么。

所以，当调用 ［self class] 时，实际先调用的是 objc_msgSend函数，第一个参数是 Son当前的这个实例，然后在 Son 这个类里面去找 - (Class)class这个方法，没有，去父类 Father里找，也没有，最后在 NSObject类中发现这个方法。而 - (Class)class的实现就是返回self的类别，故上述输出结果为 Son。

objc Runtime开源代码对- (Class)class方法的实现:

```
- (Class)class {
    return object_getClass(self);
}
```

我们对这部分代码打断点，可以发现只有在第一个 NSLog 的时候，才会命中 class：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt95oqn145j319y0u077f.jpg)

执行一个方法，会有 调用者 也会有 接收者，而使用 `NSStringFromClass`，打印的是接收者的 class

也就是说 class方法的返回值是调用这个方法的接受者; 而super只是一个编译器指示符,表示编译的时候调用objc_msgSendSuper.

## 四、进一步深入思考

```
@implementation Son : Father
- (id)init {
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

答案是Son / Son。孙源大神给的一个简短解释是这样的：

```
因为super为编译器标示符，向super发送的消息被编译成objc_msgSendSuper，但仍以self作为reveiver。
```

这个解释显然不太够，看了之后依然有点懵逼。因为，这个没有说背后的理论证据，只给了一个结论性的理由。所以有另外一个大神，在他的文章中详细解释了，这也是目前流传最广的一份答案，ChunYeah大佬的博客刨根问底 Objective－C Runtime（1）－ Self & Super。

贴一下他给出的解答中的关键部分：


>> 0. 调用 [super class]
>> 1. 会转换成objc_msgSendSuper函数
>> 2. 构造 objc_super 结构体，结构体第一个成员就是 self ，第二个成员是 (id)class_getSuperclass(objc_getClass(“Son”)) , 实际该函数输出结果为 Father。
>> 3. 去 Father这个类里去找- (Class)class，没有，然后去NSObject类去找，找到了。
>> 4. 最后内部是使用 objc_msgSend(objc_super->receiver, @selector(class))去调用，此时已经和[self class]调用相同了，故上述输出结果仍然返回 Son。


这个解释中，有一个地方的跨度非常大，就是：

```
最后内部是使用 objc_msgSend(objc_super->receiver, @selector(class))去调用
```

这个地方非常奇怪，就是本来一直都是调用的objc_msgSendSuper函数，为啥最后换了调用另一个函数。单从文章全文来看，似乎并没有做出解释。

我的猜测是，作者可能认为， objc_msgSendSuper函数在向父类中去找实现的时候，最后一直找到了NSObject才找到了 - (Class)class这个方法的实现，于是这个函数找到这个实现后，直接对这个类对象发送消息，变成了 objc_msgSend(objc_super->receiver, @selector(class))。

也就是说，用objc_msgSendSuper找方法实现，找到了，再通过objc_msgSend给调用者发送消息，重新去沿着继承链再找方法实现。这么看来，岂不是很傻，每一次调用父类方法的时候，都要遍历两遍继承链？虽然runtime有很好的方法缓存机制，但是如此遍历，肯定不是一个好的设计。况且，objc_msgSendSuper这个函数名就表示，就是已经在给父类发消息了，从代码的调用上，也可以证明。

下面就是调试代码：

```
@interface Father : NSObject
@end
  
@implementation Father
  
- (Class)class {
    return [NSValue class];
}

@end

@interface Son : Father
- (void)superClassName;
@end

@implementation Son
  
- (Class)class {
    return [NSNumber class];
}

- (void)superClassName {
    NSLog(@"Son's super Class is : %@", [super class]);
}

@end

int main(int argc, const char * argv[]) {
    Son *foo = [Son new];
    [foo superClassName];
    return 0;
}
```

这段代码就可以很好的反映，究竟是不是在调用super方法时是不是发送两次消息。

首先，在两个类的class重写方法中，打断点。

运行起来后，第一次的断点，是走Father类。因为此时实际上是objc_msgSendSuper在发送消息。继续运行，Son的断点是根本不会走的。会直接输出Son's super Class is : NSValue。

所以，为什么当父类重写- (Class)class方法时，输出就是正确的？

按照ChunYeal大佬的说法，找到方法后会变成使用 objc_msgSend(objc_super->receiver, @selector(class))去调用，那么还是输出了 Son 的类才对啊。

所以，这个例子就证明了两点：

- 1.沿着父类找实现的过程只有一次。
- 2.使用super关键字究竟能不能得到正确结果，取决于是不是调用的方法是由NSObject来实现的。

所以思路清晰了，重新捋一下。原来题目中整个过程应该是这样的

- 1. Son 对象调用 super class方法，编译器转换为objc_msgSendSuper函数发送class消息。
- 2.函数直接前往 Father 的实现中寻找，发现并没有实现。
- 3.继续沿着继承链，找到了 NSObject，有class方法实现。
- 4.直接返回结果给调用者，这个结果就是Son Class。

所以，我们就需要明确，NSObject类的class方法实现是怎样的原理。

所以，我们可以查看源文件NSObject.mm可以看到：

```
- (Class)class {
    return object_getClass(self);
}
```

继续看：

```
Class object_getClass(id obj)
{
    if (obj) return obj->getIsa();
    else return Nil;
}
```

马上接近真相：

```
objc_object::getIsa() 
{
    if (!isTaggedPointer()) return ISA();

    uintptr_t ptr = (uintptr_t)this;
    if (isExtTaggedPointer()) {
        uintptr_t slot = 
            (ptr >> _OBJC_TAG_EXT_SLOT_SHIFT) & _OBJC_TAG_EXT_SLOT_MASK;
        return objc_tag_ext_classes[slot];
    } else {
        uintptr_t slot = 
            (ptr >> _OBJC_TAG_SLOT_SHIFT) & _OBJC_TAG_SLOT_MASK;
        return objc_tag_classes[slot];
    }
}
```

再往下已经不需要看了，因为我们能明确，实际上，NSObject的- (Class)class方法实际的本质是取得isa指针。我们前面已经知道，当使用super关键字时，会转换成函数

```
objc_msgSendSuper(struct objc_super *super, SEL op, ...)
```

第一个参数是：

```
struct objc_super {
   __unsafe_unretained id receiver;
   __unsafe_unretained Class super_class;
};
```

使用clang重写文件，查看NSLog(@"%@", NSStringFromClass([super class]));对于的代码：

```
NSLog((NSString *)&__NSConstantStringImpl__var_folders_gm_0jk35cwn1d3326x0061qym280000gn_T_main_a5cecc_mi_1, NSStringFromClass(((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){ (id)self, (id)class_getSuperclass(objc_getClass("Son")) }, sel_registerName("class"))));
```

所以可以得到结论，在向父类传递消息的时候，是去父类找实现，但是消息的接收者receiver依然是self。这一点，孙源大神讲得的确正确。但是关键就在于：

`真正使得例子中返回的class为 Son 的原因，在于返回的是self的isa。`

向上寻找的过程中，并不在乎是具体的哪一个父类实现了方法。而到了NSObject中，也没有办法真的知道这个消息的接受者究竟是通过什么方式来访问isa的，所以，作为基类，就直接返回消息接受者的信息，这个在设计上就避免了很多不必要的问题。

## 五、阶段性小结

也就是说 [super class]实际上是返回 isa ，所以只要搞懂 getIsa() 的含义就可以了。

## 六、isa 指针与内存管理

关于 runtime 和 isa 指针的问题请统一看这篇文章：[架构学习自省与Runtime深析
](https://bninecoding.com/jia-gou-xue-xi-zi-sheng-yu-runtime-shen-xi.html)

## 七、最后小结

我们看懂了 getISA() 这个方法做的事情，最后可以明确的是： getISA() 从 ptr 获取 isa 时，如果发现是 objc_super ，则返回 objc_super.receiver ，也就是说这里针对 receiver 做了特殊的处理， getISA()返回的是 receiver ，而不是 superClass。

这也是为什么在 init 方法中调用 [super init] 不会死循环的原因：相当于 son 去调用了 father 的 init方法。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)