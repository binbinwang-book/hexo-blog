---
title: Objective-C 之 block
date: 2021-10-03 15:42
tags: [iOS,block]
categories: [iOS]
---

# Objective-C 之 block

## 前言

作为iOS开发，我们平日里会高频使用block，block非常重要，在学习Swift闭包时，我突然觉得可以将 Objective-C block 和 Swift闭包 一起对比学习。

如果你针对下面的问题已经有了比较深的理解，那么可以略过本篇文章：

1. block 的数据结构
2. block 的内存机制
3. block 和 weakify/strongify 的关联
4. Swift闭包和 Objective-C block的区别
5. dispatch_block_t 的应用场景

## 一、block的数据结构

## （一）block 语法解析

作为硬核派，了解block数据结构我们肯定不能Google别人的结论，我们有自己的clang工具，使用clang工具，可以将 OC 代码转成 C++ 代码。

首先，我们准备 `main.m` 这个类，类内容为：

```
// main.m 

int main() {
    return 1;
}
```

我们切到`main.m`类所在文件夹，使用指令`xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m`，发现语法解析生成的cpp代码如下`main.cpp`：

```
// main.cpp

#import <UIKit/UIKit.h>

int main() {
    void (^block)(void) = ^ {
        NSLog(@"Hello World!");
    };
    block();
    return 1;
}

```

接着我们在`main.m`中添加`block`代码：

```
// main.m 

int main() {
    void (^block)(void) = ^ {
    };
    block();
    return 1;
}
```

继续使用`clang`解析`main.m`，发现生成的cpp代码如下:

```
// main.cpp

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

        NSLog((NSString *)&__NSConstantStringImpl__var_folders_9b_w0ymsg0n3yqdlb90w49xqmz40000gn_T_main_2428cf_mi_0);
    }

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main() {
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    return 1;
}
static struct IMAGE_INFO { unsigned version; unsigned flag; } _OBJC_IMAGE_INFO = { 0, 2 };
```

去除强制转换后我们可以看出声明block的时候，block底层调用了`__main_block_iml_0`结构体，传入的参数分别是`__main_block_func_0(方法函数)`和`&__main_block_desc_0_DATA(结构体地址)`。

## （二）block Cpp 数据结构解析

### 1.__main_block_func_0 的方法代码段

```
//需要传入的参数是结构体： __main_block_impl_0
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

        NSLog((NSString *)&__NSConstantStringImpl__var_folders_9b_w0ymsg0n3yqdlb90w49xqmz40000gn_T_main_2428cf_mi_0);
    }
}
```

可以看到，这个函数体中传入了 `__cself` 和 block 中调用的方法 `NSLog`。

### 2. __main_block_desc_0_DATA 的结构体代码段

```
static struct __main_block_desc_0 {
  size_t reserved;       //作用不大，不需要理会
  size_t Block_size;     //整个block的在内存中占的字节大小
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};//计算blcok主结构体__main_block_impl_0的大小
```

总的来言，此结构体就是为了保存block结构体的大小

### 3. __main_block_impl_0 

`__main_block_impl_0` 是承载 `block` 最重要的结构，研究`__main_block_impl_0`可以从两个方面。

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

可以发现，`__main_block_impl_0 `结构中主要包含了两个结构体：

- struct __block_impl impl ：函数指针
- struct __main_block_desc_0* Desc ： block 大小等内存信息

`__block_impl`和`__main_block_desc_0`均由构建 `block`的方法传入，比如上面我们构建block传入的两个参数：`__main_block_func_0`和`__main_block_desc_0`，就是用来构建 `impl` 和 `Desc`的。

## 二、block约束问题

通过分析普通 `block` 函数的Cpp结构，我们明确了`block`函数的基本数据结构，下面我们进一步分析：为什么使用`block`时，需要有以下的关键词修饰：

- 捕获的变量需要使用 __block 才能修改
- 使用self需要用关键词 weakify、strongify 修饰

### （一）捕获的变量需要使用 __block 才能修改

```
#import "ViewController.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    __block NSInteger num = 0;
    void (^blockTest)(void) = ^ {
        num = 1;
    };
    blockTest();
}

@end
```

代码示例如上，我们可以发现，如果我们定义的 `NSInteger num` 如果不用 __block 修饰，编译器会报错：`Variable is not assignable (missing __block type specifier)`，那么我们会产生一个疑问：`block`究竟是如何捕获变量的呢？为什么我要修改的变量必须要用`__block关键词`进行修饰才能在`block`中对其进行修改？

使用指令将上述代码生成对应的 cpp 代码，内容如下：

```
static void _I_ViewController_viewDidLoad(ViewController * self, SEL _cmd) {
    ((void (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("ViewController"))}, sel_registerName("viewDidLoad"));

    __attribute__((__blocks__(byref))) __Block_byref_num_0 num = {(void*)0,(__Block_byref_num_0 *)&num, 0, sizeof(__Block_byref_num_0), 0};
    void (*blockTest)(void) = ((void (*)())&__ViewController__viewDidLoad_block_impl_0((void *)__ViewController__viewDidLoad_block_func_0, &__ViewController__viewDidLoad_block_desc_0_DATA, (__Block_byref_num_0 *)&num, 570425344));
    ((void (*)(__block_impl *))((__block_impl *)blockTest)->FuncPtr)((__block_impl *)blockTest);
}
```

我们发现和第一部分我们在 `main.m` 定义简单的 `block` 不同，这里的生成的`block`并不是`struct __main_block_impl_0`，而是`struct __ViewController__viewDidLoad_block_impl_0`。

所以我们首先可以明确的是，不同的`block`有其自己的命名规范，但后缀基本都是`_block_impl_0`。

接着我们查看 `__ViewController__viewDidLoad_block_impl_0` 的定义：

```
struct __ViewController__viewDidLoad_block_impl_0 {
  struct __block_impl impl;
  struct __ViewController__viewDidLoad_block_desc_0* Desc;
  __Block_byref_num_0 *num; // by ref
  __ViewController__viewDidLoad_block_impl_0(void *fp, struct __ViewController__viewDidLoad_block_desc_0 *desc, __Block_byref_num_0 *_num, int flags=0) : num(_num->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

与第一部分`__main_block_impl_0`相比，`__ViewController__viewDidLoad_block_impl_0`增加了 `__Block_byref_num_0 *num` 这个字段，也就是说我们在`block`中引用的字段，都会出现在`block`结构体中。

我们看到`__Block_byref_num_0 *num`后标注了`// by ref`，也就意味着block中对`num`实际是引用（不是copy），所以我们需要对`num`使用关键词`__block`将其转成`__Block_byref_num_0`引用类型。

### （二）使用self需要用关键词 weakify/strongify 修饰

#### 1. 不使用 weakify/strongify 修饰，会发生什么？

初学`block`时，我们大概率都会遇到一个问题：`block`中使用的`self`没有进行 `weakify/strongify`处理，我们也知道这样做的问题：

当`block`中使用了`self`（没有进行`weakify/strongify`声明），当执行`block`时，`self`如果已经被释放，那么在`block`中执行`self`方法应用就会 crash，因为`self`已经被释放。

如果不对`block`中使用的`self`声明`weakify/strongify`，生成的 cpp 代码会是什么情况：

```
#import "ViewController.h"

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    
    void (^blockTest)(void) = ^ {
        self.view.backgroundColor = [UIColor orangeColor];
    };
    blockTest();
}

@end
```

生成的cpp代码：

```
struct __ViewController__viewDidLoad_block_impl_0 {
  struct __block_impl impl;
  struct __ViewController__viewDidLoad_block_desc_0* Desc;
  ViewController *self;
  __ViewController__viewDidLoad_block_impl_0(void *fp, struct __ViewController__viewDidLoad_block_desc_0 *desc, ViewController *_self, int flags=0) : self(_self) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __ViewController__viewDidLoad_block_func_0(struct __ViewController__viewDidLoad_block_impl_0 *__cself) {
  ViewController *self = __cself->self; // bound by copy

        ((void (*)(id, SEL, UIColor * _Nullable))(void *)objc_msgSend)((id)((UIView *(*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("view")), sel_registerName("setBackgroundColor:"), ((UIColor * _Nonnull (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("UIColor"), sel_registerName("orangeColor")));
    }
static void __ViewController__viewDidLoad_block_copy_0(struct __ViewController__viewDidLoad_block_impl_0*dst, struct __ViewController__viewDidLoad_block_impl_0*src) {_Block_object_assign((void*)&dst->self, (void*)src->self, 3/*BLOCK_FIELD_IS_OBJECT*/);}

static void __ViewController__viewDidLoad_block_dispose_0(struct __ViewController__viewDidLoad_block_impl_0*src) {_Block_object_dispose((void*)src->self, 3/*BLOCK_FIELD_IS_OBJECT*/);}

static struct __ViewController__viewDidLoad_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __ViewController__viewDidLoad_block_impl_0*, struct __ViewController__viewDidLoad_block_impl_0*);
  void (*dispose)(struct __ViewController__viewDidLoad_block_impl_0*);
} __ViewController__viewDidLoad_block_desc_0_DATA = { 0, sizeof(struct __ViewController__viewDidLoad_block_impl_0), __ViewController__viewDidLoad_block_copy_0, __ViewController__viewDidLoad_block_dispose_0};

static void _I_ViewController_viewDidLoad(ViewController * self, SEL _cmd) {
    ((void (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("ViewController"))}, sel_registerName("viewDidLoad"));

    void (*blockTest)(void) = ((void (*)())&__ViewController__viewDidLoad_block_impl_0((void *)__ViewController__viewDidLoad_block_func_0, &__ViewController__viewDidLoad_block_desc_0_DATA, self, 570425344));
    ((void (*)(__block_impl *))((__block_impl *)blockTest)->FuncPtr)((__block_impl *)blockTest);
}
```

关键语句是：

```
static void __ViewController__viewDidLoad_block_func_0(struct __ViewController__viewDidLoad_block_impl_0 *__cself) {
  ViewController *self = __cself->self; // bound by copy

        ((void (*)(id, SEL, UIColor * _Nullable))(void *)objc_msgSend)((id)((UIView *(*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("view")), sel_registerName("setBackgroundColor:"), ((UIColor * _Nonnull (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("UIColor"), sel_registerName("orangeColor")));
    }
```

`bound by copy` ，使用 `copy` 的方式进行绑定，我们知道`copy`意味着浅拷贝，被引用的对象引用计数会+1，那么这样就会出问题：

>> 0. `self` 中定义了 `block`，相当于`self`持有了`block`
>> 1. 同时 `block` 中又持有了 `self`
>> 2. 导致循环引用，该释放的对象无法被释放，内存泄露

为了验证上面所说的`循环引用导致无法回收`的情况，我们来模拟一个场景：

```
@implementation BNDestroyDemoView

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"BNDestroyDemoView initWithFrame:%@",self);
        });
    }
    return self;
}

@end

```

我们新建了一个类`BNDestroyDemoView.h`，这个类被创建后会被立刻置nil:

```
@interface ViewController ()

@property (nonatomic, strong) BNDestroyDemoView *demoView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    self.demoView = [[BNDestroyDemoView alloc] initWithFrame:CGRectMake(0, 0, 50, 50)];
    self.demoView = nil;
}


@end
```

这个类会延迟3秒执行`dispatch_after`中的block内容，即使我们已经将 `self.demoView`置nil，3秒后我们依旧可以看到如下的日志打印，表明`self`并没有被系统回收：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gv25f9v5nhj612o06mgmm02.jpg)

#### 2. weakify/strongify 修饰，是进行深拷贝吗？

通过上面的分析，我们明白一个道理：使用`block`时要避免产生`循环引用`，既然浅拷贝会导致引用计数+1。

既然如上的浅拷贝逻辑会导致循环引用，我们有什么办法解决循环引用呢？

- 深拷贝
- 弱引用，强使用

深拷贝的方法是将`self`的内存直接拷贝一份，不对原`self`的引用计数新增，这种方法首先从开销上会比较大，而且有时`self`如果被重置为nil，我们的目标就是不执行`self`的方法，而不是执行深拷贝后的`self`方法。

所以那就只有使用`弱引用，强使用`的方法了，这种方法在iOS开发中是一种通用的解决方案，在 `Runloop循环引用Timer`、`YYAsyncLabel`等技术方案中都有使用，下面进行具体的阐述。

`弱引用`的意思是：我传入 `block` 中的 `self` 通过 `weak`进行修饰，不增加 `self` 的引用计数

`强使用`的意思是：我在执行`block`方法体期间，需要将`弱引用`的`self`改为`强引用`，避免在执行`block`期间`self`被回收。

对应的代码实现如下：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    __weak __typeof(self) weakSelf  = self;
    void (^block)(void) = ^ {
        __strong __typeof(self) strongSelf = weakSelf;
        NSLog(@"Hello World!");
    };
    block();
}
```

于此我们便解决了使用`block`会导致循环引用的问题，但继而又产生了一个问题：

`强引用`的`self`有可能为nil吗？

答案是：可能。如果在执行`block`之前，`self`就已经被回收，因为`block`在执行前对`self`是弱引用，所以`self`是有可能变为nil的。

```
@implementation BNDestroyDemoView

- (instancetype)initWithFrame:(CGRect)frame {
    if (self = [super initWithFrame:frame]) {
        __weak __typeof(self) weakSelf  = self;
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            __strong __typeof(self) strongSelf = weakSelf;
            NSLog(@"BNDestroyDemoView initWithFrame:%@",@[strongSelf]);
        });
    }
    return self;
}

@end
```

上面这段代码是非常不健壮的，如果`BNDestroyDemoView`在执行 `block` 之前被系统回收，就会导致crash：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gv25u62n9cj61r40g6tea02.jpg)

## 三、dispatch_block_t 的应用场景

通常我们写一个不带参数的块回调函数是这样写的：

```
在 . h 头文件中
typedef void (^leftBlockAction)(); // 定义类型
-(void)leftButtonAction:(leftBlockAction)leftBlock; // 在定义一个回调函数：

在.m 文件中：
-(void)leftButtonAction:(leftBlockAction)leftBlock{
	leftBlock();
}

```

使用`dispatch_block_t ` 只要在.h 头文件定义属性方法

`@property (nonatomic,copy) dispatch_block_t leftBlockAction;`

在.m文件 调用的方法里调用

```
if (self.leftBlockAction) {
    self.leftBlockAction();
}
```

在另个模块里直接:
```
MyAlertView *alert = [[MyAlertView alloc]init];
alert.leftBlockAction = ^() {

    NSLog(@"left button clicked");
};
```

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)