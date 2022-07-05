---
title: iOS内存管控实战（中）-分析工具篇
date: 2022-06-05 22:32
tags: [iOS,内存]
categories: [iOS]
---

# iOS内存管控实战（中）-分析工具篇

因文章单篇过长，按照 原理、分析工具 和 实战 拆分成上、中、下三部分，点击阅读。
- [iOS内存管控实战（上）—原理篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-shang-yuan-li-pian.html)
- [iOS内存管控实战（中）-分析工具篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-zhong-fen-xi-gong-ju-pian.html)
- [iOS内存管控实战（下）—实战篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-xia-shi-zhan-pian.html)

## 二、内存分析工具

### （一）分析工具一览

关于内存占用情况、内存泄漏，我们都有一系列方法进行分析检测：

- Xcode memory gauge：在 Xcode 的 Debug navigator 中，可以粗略查看内存占用的情况；
- Instrument - Allocations：可以查看虚拟内存占用、堆信息、对象信息、调用栈信息，VM Regions 信息等。可以利用这个工具分析内存，并针对地进行优化；
- Instrument - Leaks：用于检测内存泄漏；
- MLeaksFinder：通过判断 UIViewController 被销毁后其子 view 是否也都被销毁，可以在不入侵代码的情况下检测内存泄漏；
- Instrument - VM Tracker：可以查看内存占用信息，查看各类型内存的占用情况，比如 dirty memory 的大小等等，可以辅助分析内存过大、内存泄漏等原因；
- Instrument - Virtual Memory Trace：有内存分页的具体信息；
- Memory Resource Exceptions：从 Xcode 10 开始，内存占用过大时，调试器能捕获到 EXC_RESOURCE RESOURCE_TYPE_MEMORY 异常，并断点在触发异常抛出的地方；
- Xcode Memory Debugger：Xcode 中可以直接查看所有对象间的相互依赖关系，可以非常方便的查找循环引用的问题，同时，还可以将这些信息导出为 memgraph 文件；
memgraph + 命令行指令：结合上一步输出的 memgraph 文件，可以通过一些指令来分析内存情况。vmmap 可以打印出进程信息，以及 VMRegions 的信息等，结合 grep 可以查看指定 VMRegion 的信息。leaks 可追踪堆中的对象，从而查看内存泄漏、堆栈信息等。heap 会打印出堆中所有信息，方便追踪内存占用较大的对象。malloc_history 可以查看 heap 指令得到的对象的堆栈信息，从而方便地发现问题。总结：malloc_history ===> Creation；leaks ===> Reference；heap & vmmap ===> Size。

### （二）Xcode memory debugger 和 指令操作

相比 `Instrument` 全而大的工具，`Xcode memory debugger` 在可视化内存泄露上使用起来更简单，下面对 `Xcode memory debugger` 的使用进行举例：

```
#import "BNTestViewController.h"

@interface BNTestViewController ()

@end

@implementation BNTestViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationBackgroundRefreshStatusDidChangeNotification object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {
        self.view.backgroundColor = [UIColor redColor];
    }];
}

@end
```

我们在 `BNTestViewController.h ` 这个类中使用的block强引用了自己，打开`BNTestViewController`界面再退出，预期`BNTestViewController`将会出现内存泄露，我们点击如下入口打开`Memory Debugger`：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdm54o5cj20oh0g2dhb.jpg)

接着我们可以看到有一个block强引用了 `BNTestViewController` ：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xdr0a891j21c00me41r.jpg)

无论是什么情况，block强引用ViewController都是不应该出现的，接着我们需要定位是哪个block强引用了`BNTestViewController`，想看block所对应的实现代码时你只需要在lldb控制台输入如下信息：

```
(lldb) dis -s *(void**)(0x6000032e9740+16)
BNTestMemoryLeakDemo`__35-[BNTestViewController viewDidLoad]_block_invoke:
    0x1025a1dac <+0>:  sub    sp, sp, #0x50
    0x1025a1db0 <+4>:  stp    x29, x30, [sp, #0x40]
    0x1025a1db4 <+8>:  add    x29, sp, #0x40
    0x1025a1db8 <+12>: str    x0, [sp]
    0x1025a1dbc <+16>: stur   x0, [x29, #-0x8]
    0x1025a1dc0 <+20>: sub    x0, x29, #0x10
    0x1025a1dc4 <+24>: str    x0, [sp, #0x18]
    0x1025a1dc8 <+28>: mov    x8, #0x0
```

上述指令中 dis -s 地址  的作用是用来反汇编某个地址所对应符号信息以及开始一部分的汇编实现。

命令中而后面的0x600002f51110 则是Block对象的地址，这里加16的意思是因为Block对象的内部偏移16个字节的位置就是Block对象所保存的执行代码的函数地址。 所以通过这个指令就可以轻松的知道是哪个Block对象强持有了对象而不会被释放了。

当然需要注意的是，这个方法只能查看无参数传入的block地址，当无法使用该命令时，可以检索泄露类本身的block方法，还是比较好查看的。


### （三）Facebook循环引用检测框架：FBRetainCycleDetector

#### 1. FBRetainCycleDetector 优势

[FBRetainCycleDetector](https://github.com/facebook/FBRetainCycleDetector) 是由 Facebook开源的一套检测循环引用的库，官方描述是：

```
Retain cycles are one of the most common ways of creating memory leaks. It's incredibly easy to create a retain cycle, and tends to be hard to spot it. The goal of FBRetainCycleDetector is to help find retain cycles at runtime. The features of this project were influenced by Circle.
```

使用 `FBRetainCycleDetector` 和上面提到的Xcode提供的内存泄露检测软件不同的是：`FBRetainCycleDetector`可以在代码层面就检测到是否存在泄露的可能，无需复现内存情况就可以找到泄露的代码。

比如这一段代码中我们用一个block强引用自己：

```

@interface ViewController ()
{
    void (^_handlerBlock)();
}

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    _handlerBlock = ^{
        NSLog(@"%@", self);
    };
    
    FBRetainCycleDetector *detector = [FBRetainCycleDetector new];
    [detector addCandidate:self];
    NSSet *retainCycles = [detector findRetainCycles];
    NSLog(@"%@", retainCycles);
}

@end
```

使用`FBRetainCycleDetector`检测会得到如下内容：

```
2022-06-05 15:47:48.915720+0800 BNTestFacebookCycleDemo[91381:386714] {(
        (
        "-> ViewController ",
        "-> _handlerBlock -> __NSMallocBlock__ "
    )
)}
```

而如果你使用类似`Xcode memory Debugger`一类的工具，只有带 `ViewController`出现了内存泄露，才能在内存图表（Memory Graph）上看出来。 :(

#### 2. FBRetainCycleDetector原理先备知识

##### （1）有向图

在数学上，一个图（Graph）是表示物件与物件之间的关系的方法，是图论的基本研究对象。一个图看起来是由一些小圆点（称为顶点或结点）和连结这些圆点的直线或曲线（称为边）组成的。如果给图的每条边规定一个方向，那么得到的图称为有向图，其边也称为有向边。在有向图中，与一个节点相关联的边有出边和入边之分，而与一个有向边关联的两个点也有始点和终点之分。相反，边没有方向的图称为无向图。

图的遍历方法有深度优先搜索法和广度（宽度）优先搜索法。

##### （2）深度优先搜索法和广度（宽度）优先搜索法

深度优先搜索法是树的先根遍历的推广，它的基本思想是：从图G的某个顶点v0出发，访问v0，然后选择一个与v0相邻且没被访问过的顶点vi访问，再从vi出发选择一个与vi相邻且未被访问的顶点vj进行访问，依次继续。如果当前被访问过的顶点的所有邻接顶点都已被访问，则退回到已被访问的顶点序列中最后一个拥有未被访问的相邻顶点的顶点w，从w出发按同样的方法向前遍历，直到图中所有顶点都被访问。

图的广度优先搜索是树的按层次遍历的推广，它的基本思想是：首先访问初始点vi，并将其标记为已访问过，接着访问vi的所有未被访问过的邻接点vi1,vi2,…, vi t，并均标记已访问过，然后再按照vi1,vi2,…, vi t的次序，访问每一个顶点的所有未被访问过的邻接点，并均标记为已访问过，依次类推，直到图中所有和初始点vi有路径相通的顶点都被访问过为止。

#### 3. FBRetainCycleDetector 检测的具体原理

简单来说它的工作原理就是从传入的目标对象开始查找其强引用的对象列表，在这个强引用对象列表里继续查找强引用对象，默认查找 10 层。最后在整个有向图中应用 DFS 算法查找环。如果有向图存在环，则说明目标对象存在循环引用。

首先FBRetainCycleDetector主要用到了runtime，可以获取强引用的成员变量。而swift默认并不能获取这些，所以只能作用在oc上。

关键步骤： 

- FBGetObjectStrongReferences： 获取到对象的强引用成员变量列表，要递归父类调用FBGetStrongReferencesForClass获取。
- FBGetStrongReferencesForClass： FBGetClassReferences获取到成员变量列表，先采用 wrapper.type != FBUnknownType 过滤掉基础类型，再过滤选择了强引用类型
- FBGetClassReferences:  获取到Class的成员变量列表。
- 最后得到的数据结构是：一个有向图，图之间是强引用的对象的连接。
- 然后就遍历有向图，判断有没有环。

检测的核心代码如下：
```
  NSMutableSet<NSArray<FBObjectiveCGraphElement *> *> *retainCycles = [NSMutableSet new];
  FBNodeEnumerator *wrappedObject = [[FBNodeEnumerator alloc] initWithObject:graphElement];

  NSMutableArray<FBNodeEnumerator *> *stack = [NSMutableArray new];
  NSMutableSet<FBNodeEnumerator *> *objectsOnPath = [NSMutableSet new];

  [stack addObject:wrappedObject];

  while ([stack count] > 0) {
    @autoreleasepool {

      FBNodeEnumerator *top = [stack lastObject];

      if (![objectsOnPath containsObject:top]) {
        if ([_objectSet containsObject:@([top.object objectAddress])]) {
          [stack removeLastObject];
          continue;
        }
        [_objectSet addObject:@([top.object objectAddress])];
      }

      [objectsOnPath addObject:top];

        //获取子节点迭代器
      FBNodeEnumerator *firstAdjacent = [top nextObject];
      if (firstAdjacent) {
        //有子节点
        BOOL shouldPushToStack = NO;
        //当前链路上已存在当前子节点
        if ([objectsOnPath containsObject:firstAdjacent]) {
         
          NSUInteger index = [stack indexOfObject:firstAdjacent];
          NSInteger length = [stack count] - index;

          if (index == NSNotFound) {
            shouldPushToStack = YES;
          } else {
            //构建环结构
            NSRange cycleRange = NSMakeRange(index, length);
            NSMutableArray<FBNodeEnumerator *> *cycle = [[stack subarrayWithRange:cycleRange] mutableCopy];
            [cycle replaceObjectAtIndex:0 withObject:firstAdjacent];
            [retainCycles addObject:[self _shiftToUnifiedCycle:[self _unwrapCycle:cycle]]];
          }
        } else {
          shouldPushToStack = YES;
        }

        if (shouldPushToStack) {
          if ([stack count] < stackDepth) {
            [stack addObject:firstAdjacent];
          }
        }
      } else {
         //无子节点
        [stack removeLastObject];
        [objectsOnPath removeObject:top];
      }
    }
  }
  return retainCycles;
}
```

#### 4. 从 FBRetainCycleDetector 原理中可以学到的

##### （1）根据属性Ivar.typeEncoding区分 struct、block 和 object

```
- (FBType)_convertEncodingToType:(const char *)typeEncoding
{
  if (typeEncoding[0] == '{') {
    return FBStructType;
  }

  if (typeEncoding[0] == '@') {
    // It's an object or block

    // Let's try to determine if it's a block. Blocks tend to have
    // @? typeEncoding. Docs state that it's undefined type, so
    // we should still verify that ivar with that type is a block
    if (strncmp(typeEncoding, "@?", 2) == 0) {
      return FBBlockType;
    }

    return FBObjectType;
  }

  return FBUnknownType;
}
```

##### （2）获取一个对象强引用列表

普通OC对象会通过哪些方式强持用其他对象呢？

- ivar 成员变量
- 集合类中的元素，比如 NSArray、NSCollection 中的元素
- associatedObject

注意：通过这三种方式添加的引用都有可能是 弱引用 ，实际处理时需要将弱引用的情况排除掉。

##### a. ivar 成员变量

在OC中，我们可以直接添加一个 ivar ，也可以通过声明 property 的方式添加 ivar 。 ivar 和 property 都能够以弱引用的方式添加。

```
@interface DemoClass () {
    NSObject *_strongIvar;
    __weak NSObject *_weakIvar;
}
@property(nonatomic, strong) NSObject *strongProperty;
@property(nonatomic, weak) NSObject *weakProperty;
@end
```

如何获取所有 强引用 的 ivar 呢？

- 1. 找出所有 ivar
- 2. 筛选强引用的 ivar

##### 1. 找出所有 ivar

OC Runtime 提供了 class_copyIvarList 方法，该方法可以直接返回 Class 的 ivar 列表。

FBRetainCycleDetector 中对应的源码为：（有删减）

```
NSArray<id<FBObjectReference>> *FBGetClassReferences(Class aCls) {
  NSMutableArray<id<FBObjectReference>> *result = [NSMutableArray new];
  // 获取 ivar 列表
  unsigned int count;
  Ivar *ivars = class_copyIvarList(aCls, &count);
  // 封装为 FBIvarReference 对象
  for (unsigned int i = 0; i < count; ++i) {
    Ivar ivar = ivars[i];
    FBIvarReference *wrapper = [[FBIvarReference alloc] initWithIvar:ivar];
    [result addObject:wrapper];
  }
  free(ivars);
  return [result copy];
}
```

##### 2. 筛选强引用的 ivar

ivar 列表里包含了所有强引用和弱引用的 ivar 。比如对于上面示例代码中的 DemoClass 对象， class_copyIvarList 返回的结果包含了：

- _strongIvar
- _weakIvar
- _strongProperty
- _weakProperty

如何才能筛选出强引用的对象（也就是 _strongIvar 和 _strongProperty ）呢？

OC Runtime中还提供了 class_getIvarLayout 方法用于获取所有强引用 ivar 的布局信息；还有一个对应的 class_getWeakIvarLayout 方法用于获取所有弱引用 ivar 的布局信息。

```
/** 
 * Returns a description of the \c Ivar layout for a given class.
 * 
 * @param cls The class to inspect.
 * 
 * @return A description of the \c Ivar layout for \e cls.
 */
OBJC_EXPORT const uint8_t * _Nullable
class_getIvarLayout(Class _Nullable cls)
    OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);

/** 
 * Returns a description of the layout of weak Ivars for a given class.
 * 
 * @param cls The class to inspect.
 * 
 * @return A description of the layout of the weak \c Ivars for \e cls.
 */
OBJC_EXPORT const uint8_t * _Nullable
class_getWeakIvarLayout(Class _Nullable cls)
    OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0, 2.0);
```

所以，拿到 ivar 列表之后，我们只需遍历一遍所有的 ivar ，判断该 ivar 对应的布局位置是否为强引用即可。

FBRetainCycleDetector 中对应的源码为：（有删减）

```
static NSArray<id<FBObjectReference>> *FBGetStrongReferencesForClass(Class aCls) {
  // 获取 ivar 列表
  NSArray<id<FBObjectReference>> *ivars = FBGetClassReferences(aCls);
  // 获取强引用 ivar 布局信息
  const uint8_t *fullLayout = class_getIvarLayout(aCls);
  if (!fullLayout) {
    return @[];
  }
  NSUInteger minimumIndex = FBGetMinimumIvarIndex(aCls);
  NSIndexSet *parsedLayout = FBGetLayoutAsIndexesForDescription(minimumIndex, fullLayout);

  // 遍历 ivar，确认是否属于强引用
  NSArray<id<FBObjectReference>> *filteredIvars =
  [ivars filteredArrayUsingPredicate:[NSPredicate predicateWithBlock:^BOOL(id<FBObjectReference> evaluatedObject,
                                                                           NSDictionary *bindings) {
    return [parsedLayout containsIndex:[evaluatedObject indexInIvarLayout]];
  }]];

  return filteredIvars;
}
```

总结： 通过 class_copyIvarList 和 class_getIvarLayout ，可以获取所有的强引用 ivar 。

##### b. 集合类中的元素

我们平常使用的 NSArray 和 NSDictionary 默认对元素都是强引用，对于这些集合类，直接遍历其元素就能够获取所有其强引用的所有对象。

但是现实总是比预想的要复杂。还有一些集合类，比如 NSMapTable 等，可以自定义对元素的引用类型，可以分别设置 key 和 value 使用弱引用还是强引用。

```
// NSMapTable.h
+ (id)mapTableWithStrongToStrongObjects;
+ (id)mapTableWithWeakToStrongObjects;
+ (id)mapTableWithStrongToWeakObjects;
+ (id)mapTableWithWeakToWeakObjects;
```

有没有办法能知道这些集合类的对元素的引用到底是弱引用还是强引用呢？
当然可以。在 NSMapTable 头文件中，我们能找到 keyPointerFunctions 和 valuePointerFunctions 这两个方法，通过这两个方法，我们就能知道 NSMapTable 对 key 和 value 到底是弱引用还是强引用了。

```
// NSMapTable.h
/* return an NSPointerFunctions object reflecting the functions in use.  This is a new autoreleased object that can be subsequently modified and/or used directly in the creation of other pointer "collections". */
@property (readonly, copy) NSPointerFunctions *keyPointerFunctions;
@property (readonly, copy) NSPointerFunctions *valuePointerFunctions;
```

FBRetainCycleDetector 中对应的源码为：（有删减）

```
// NSArray/NSDictionary/NSMapTable 等集合类均遵循 NSFastEnumeration 协议
if ([aCls conformsToProtocol:@protocol(NSFastEnumeration)]) {
  // key 是否为强引用
  BOOL retainsKeys = [self _objectRetainsEnumerableKeys];
  // value 是否为强引用
  BOOL retainsValues = [self _objectRetainsEnumerableValues];
  BOOL isKeyValued = NO;
  if ([aCls instancesRespondToSelector:@selector(objectForKey:)]) {
    isKeyValued = YES;
  }
  // ...  
  for (id subobject in self.object) {
    if (retainsKeys) {
      // key 为强引用，获取所有 key
      FBObjectiveCGraphElement *element = FBWrapObjectGraphElement(self, subobject, self.configuration);
    }
    if (isKeyValued && retainsValues) {
      // value 为强引用，获取所有value
      FBObjectiveCGraphElement *element = FBWrapObjectGraphElement(self, [self.object objectForKey:subobject], self.configuration);
    }
  }
  // ...
}
```

具体判断 key 和 value 是否为强引用的代码为：

```
//  是否强引用 value
- (BOOL)_objectRetainsEnumerableValues
{
  if ([self.object respondsToSelector:@selector(valuePointerFunctions)]) {
    NSPointerFunctions *pointerFunctions = [self.object valuePointerFunctions];
    if (pointerFunctions.acquireFunction == NULL) {
      return NO;
    }
    if (pointerFunctions.usesWeakReadAndWriteBarriers) {
      return NO;
    }
  } 
  // 默认为强引用（如 NSArray）
  return YES;
}

// 是否强引用key
- (BOOL)_objectRetainsEnumerableKeys
{
  if ([self.object respondsToSelector:@selector(pointerFunctions)]) {
    // NSHashTable and similar
    // If object shows what pointer functions are used, lets try to determine
    // if it's not retaining objects
    NSPointerFunctions *pointerFunctions = [self.object pointerFunctions];
    if (pointerFunctions.acquireFunction == NULL) {
      return NO;
    }
    if (pointerFunctions.usesWeakReadAndWriteBarriers) {
      // It's weak - we should not touch it
      return NO;
    }
  }

  if ([self.object respondsToSelector:@selector(keyPointerFunctions)]) {
    NSPointerFunctions *pointerFunctions = [self.object keyPointerFunctions];
    if (pointerFunctions.acquireFunction == NULL) {
      return NO;
    }
    if (pointerFunctions.usesWeakReadAndWriteBarriers) {
      return NO;
    }
  }
  // 默认为强引用（如 NSDictionary）
  return YES;
}
```

总结： 通过遍历集合元素（包括 key 和 value ），并判断其是否为强引用，可以获取集合类强引用的所有元素。

##### c. associatedObject

除了上述两种常规的引用类型，在OC中我们还可以通过 objc_setAssociatedObject 在运行时为OC对象动态添加引用对象。同样，通过这种方式添加的对象也不一定是强引用对象。

```
/** 
 * Sets an associated value for a given object using a given key and association policy.
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * @param value The value to associate with the key key for object. Pass nil to clear an existing association.
 * @param policy The policy for the association. For possible values, see “Associative Object Behaviors.”
 * 
 * @see objc_setAssociatedObject
 * @see objc_removeAssociatedObjects
 */
OBJC_EXPORT void
objc_setAssociatedObject(id _Nonnull object, const void * _Nonnull key,
                         id _Nullable value, objc_AssociationPolicy policy)
    OBJC_AVAILABLE(10.6, 3.1, 9.0, 1.0, 2.0);
```

其中的 policy 用于设置引用类型，有如下取值：

```
/* Associative References */

/**
 * Policies related to associative references.
 * These are options to objc_setAssociatedObject()
 */
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};
```

显然，OBJC_ASSOCIATION_RETAIN_NONATOMIC 和 OBJC_ASSOCIATION_RETAIN 为强引用类型。

由于 objc_setAssociatedObject 是在运行时为OC对象动态添加引用，我们需要hook掉 objc_setAssociatedObject 方法调用，将运行时添加的强引用对象记录下来。

怎么hook呢？ objc_setAssociatedObject 为C方法，只能派 fishhook 上场了。

FBRetainCycleDetector 中对应的源码为：

```
// FBAssociationManager.m
+ (void)hook
{

  std::lock_guard<std::mutex> l(*FB::AssociationManager::hookMutex);
  rcd_rebind_symbols((struct rcd_rebinding[2]){
    {
      "objc_setAssociatedObject",
      (void *)FB::AssociationManager::fb_objc_setAssociatedObject,
      (void **)&FB::AssociationManager::fb_orig_objc_setAssociatedObject
    },
    {
      "objc_removeAssociatedObjects",
      (void *)FB::AssociationManager::fb_objc_removeAssociatedObjects,
      (void **)&FB::AssociationManager::fb_orig_objc_removeAssociatedObjects
    }}, 2);
  FB::AssociationManager::hookTaken = true;

}
```

#### 5. FBRetainCycleDetector 不足

通过对 FBRetainCycleDetector 源码的解析，我们可以明白 FBRetainCycleDetector 检测是否有循环引用实际是一个开销相当大的事情，但FBRetainCycleDetector的循环引用的检测逻辑做得确实很精确，当我们怀疑一个对象可能出现循环引用时，交给 FBRetainCycleDetector 可以很方便地帮我们定位有问题的代码。

那这时我们就可以考虑召唤一个哨兵，哨兵负责侦探可能会内存泄露的对象，然后具体哪里泄露交给 FBRetainCycleDetector 去处理，这里要召唤的哨兵就是 MLeaksFinder。

### （四）轻量级的内存泄露检测框架：MLeaksFinder

MLeaksFinder 提供了内存泄露检测更好的解决方案。只需要引入 MLeaksFinder，就可以自动在 App 运行过程检测到内存泄露的对象并立即提醒，无需打开额外的工具，也无需为了检测内存泄露而一个个场景去重复地操作。MLeaksFinder 目前能自动检测 UIViewController 和 UIView 对象的内存泄露，而且也可以扩展以检测其它类型的对象。

`MLeaksFinder`的设计原理很简单，MLeaksFinder 一开始从 UIViewController 入手。我们知道，当一个 UIViewController 被 pop 或 dismiss 后，该 UIViewController 包括它的 view，view 的 subviews 等等将很快被释放（除非你把它设计成单例，或者持有它的强引用，但一般很少这样做）。于是，我们只需在一个 ViewController 被 pop 或 dismiss 一小段时间后，看看该 UIViewController，它的 view，view 的 subviews 等等是否还存在。

具体的方法是，为基类 NSObject 添加一个方法 -willDealloc 方法，该方法的作用是，先用一个弱指针指向 self，并在一小段时间(3秒)后，通过这个弱指针调用 -assertNotDealloc，而 -assertNotDealloc 主要作用是直接中断言。

```
- (BOOL)willDealloc {
    __weak id weakSelf = self;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [weakSelf assertNotDealloc];
    });
    return YES;
}
- (void)assertNotDealloc {
     NSAssert(NO, @“”);
}
```

这样，当我们认为某个对象应该要被释放了，在释放前调用这个方法，如果3秒后它被释放成功，weakSelf 就指向 nil，不会调用到 -assertNotDealloc 方法，也就不会中断言，如果它没被释放（泄露了），-assertNotDealloc 就会被调用中断言。这样，当一个 UIViewController 被 pop 或 dismiss 时（我们认为它应该要被释放了），我们遍历该 UIViewController 上的所有 view，依次调 -willDealloc，若3秒后没被释放，就会中断言。

设计理念可行，但落地到应用中有7个问题需要解决：

- 1. 不入侵开发代码

这里使用了 AOP 技术，hook 掉 UIViewController 和 UINavigationController 的 pop 跟 dismiss 方法，关于如何 hook，请参考 Method Swizzling。

- 2. 遍历相关对象

在实际项目中，我们发现有时候一个 UIViewController 被释放了，但它的 view 没被释放，或者一个 UIView 被释放了，但它的某个 subview 没被释放。这种内存泄露的情况很常见，因此，我们有必要遍历基于 UIViewController 的整棵 View-ViewController 树。我们通过 UIViewController 的 presentedViewController 和 view 属性，UIView 的 subviews 属性等递归遍历。对于某些 ViewController，如 UINavigationController，UISplitViewController 等，我们还需要遍历 viewControllers 属性。

- 3. 构建堆栈信息

需要构建 View-ViewController stack 信息以告诉开发者是哪个对象没被释放。在递归遍历 View-ViewController 树时，子节点的 stack 信息由父节点的 stack 信息加上子结点信息即可。

- 4. 例外机制

对于有些 ViewController，在被 pop 或 dismiss 后，不会被释放（比如单例），因此需要提供机制让开发者指定哪个对象不会被释放，这里可以通过重载上面的 -willDealloc 方法，直接 return NO 即可。

- 5. 特殊情况

对于某些特殊情况，释放的时机不大一样（比如系统手势返回时，在划到一半时 hold 住，虽然已被 pop，但这时还不会被释放，ViewController 要等到完全 disappear 后才释放），需要做特殊处理，具体的特殊处理视具体情况而定。

- 6. 系统View

某些系统的私有 View，不会被释放（可能是系统 bug 或者是系统出于某些原因故意这样做的，这里就不去深究了），因此需要建立白名单

- 7. 手动扩展

MLeaksFinder目前只检测 ViewController 跟 View 对象。为此，MLeaksFinder 提供了一个手动扩展的机制，你可以从 UIViewController 跟 UIView 出发，去检测其它类型的对象的内存泄露。如下所示，我们可以检测 UIViewController 底下的 View Model：

```
- (BOOL)willDealloc {
    if (![super willDealloc]) {
        return NO;
    }
    MLCheck(self.viewModel);
    return YES;
}
```


## 参考文章

- [WWDC 2018：iOS 内存深入研究](https://juejin.cn/post/6844903621276991502#heading-9)
- [iOS调试Block引用对象无法被释放的一个小技巧](https://cloud.tencent.com/developer/article/1508382)
- [iOS之深入解析Memory内存](https://blog.csdn.net/Forever_wj/article/details/120578784)
- [MLeaksFinder：精准 iOS 内存泄露检测工具](https://wereadteam.github.io/2016/02/22/MLeaksFinder/)
- [OS X Mavericks 中的内存压缩技术到底有多强大？](https://www.zhihu.com/question/21775223)
- [FBRetainCycleDetector + MLeaksFinder 阅读](https://www.jianshu.com/p/76250de94b93)
- [获取OC对象的所有强引用对象](https://blog.jerrychu.top/2021/01/10/%E8%8E%B7%E5%8F%96OC%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%89%80%E6%9C%89%E5%BC%BA%E5%BC%95%E7%94%A8%E5%AF%B9%E8%B1%A1/)
- [WWDC2018 图像最佳实践](https://juejin.cn/post/6844903618429059086)

文章首发：[问我社区](http://www.wenwoha.com/)