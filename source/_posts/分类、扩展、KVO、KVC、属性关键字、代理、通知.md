---
title: 分类、扩展、KVO、KVC、属性关键字、代理、通知
date: 2021-08-01 22:32
tags: [Object-C]
categories: [iOS]
---

# 分类、扩展、KVO、KVC、属性关键字、代理、通知

## 一、分类（Category）

### （一）分类在业务上的应用

- 声明私有方法
- 分解体积庞大的类文件
- 把 Framework 的私有方法分开

### （二）Category 的特点/ Category和Extension的区别

#### 1. runtime 生成

当我们编写完 Category 文件，编译的过程并没有把 Category 的内容添加到 宿主类 上面。而是在 Runtime 阶段，通过 runtime 将 Category 的内容添加到 宿主类 上面。

`runtime生成`这个特性是 Category 最大的特点，同时也是 Category 和 Extension 最大的区别。

#### 2. 可以为 系统类 添加 Category

比如每个公司基本都有针对 UIView 获取坐标的 Category 方法，可以直接通过 .x / .y 进行属性访问。

Tips：Object-C不支持给系统类添加Extension，实际上也没必要给系统类添加 Extension。

### （三）Category 可以添加哪些内容？

- 实例方法：-(void)function
- 类方法：+(void)function
- 协议：protocol
- 属性：property

注意了，如果使用 Category 添加了 property，实际上只是声明了对应的 getter 方法 和 setter 方法，并没有为我们在 Category 中生成实例变量。

可以通过 关联对象（association_object）的方式来添加 实例变量。

### （四）Category 的源码实现

#### 1. 数据结构

源码来源： OC runtime

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt0nwxgcg4j30sh0jg0uq.jpg)

- Name：Category 的名称
- Cls：Category 所属类，也即 宿主类
- instanceMethods：实例方法列表
- classMethods：类方法列表
- protocol：协议
- instanceProperties：属性

#### 2. Category 的编写特性

- Category 添加的方法可以"覆盖"宿主类的同名方法
- Category 中同名方法谁能生效取决于编译顺序
- 名字相同的 Category 会引起编译报错

（1）对一个宿主类添加两个Category，编译两个同名方法，哪个方法会生效？

最后编译的Category中的方法会生效，前面的会被覆盖掉。

（2）Category 添加的方法可以"覆盖"宿主类的同名方法

这里的"覆盖"加了引号，原因是效果上方法会覆盖，但实际上宿主类的同名方法在内存中仍然存在。

#### 3. 将 Category attach 到 宿主类 的源码

```
static void remethodizeClass(Class cls) {
    category_list *cats;
    bool isMeta;
    runtimeLock.assertLocked();
    // 我们只分析分类当中实例方法添加的逻辑 
    // 因此在这里我们假设 isMeta = NO
    isMeta = cls->isMetaClass();
    // Re-methodizing: check for more categories
    // 获取 cls 中未完成整合的所有分类
    if ((cats = unattachedCategoriesForClass(cls, false /*not realizing*/))) {
        if (PrintConnecting) {
            _objc_inform("CLASS: attaching categories to class '%s' %s",
        }
        cls->nameForLogging(), isMeta ? "(meta)" : "");
        attachCategories(cls, cats, true /*flush caches*/);
        free(cats);
    }
}

static void attachCategories(Class cls, category_list *cats, bool flush_caches) {
    if (!cats)
        return;
    if (PrintReplacedMethods)
        printReplacements(cls, cats);
    
    bool isMeta = cls->isMetaClass();
    
    /* mlists 是个二维数组
     [[method_t,method_t,...],[method_t],[method_t,method_t,method_t],...]
     */
    method_list_t **mlists = (method_list_t **)malloc(cats->count * sizeof(*mlists));
    property_list_t **proplists = (property_list_t **)malloc(cats->count * sizeof(*proplists));
    protocol_list_t **protolists = (protocol_list_t **)malloc(cats->count * sizeof(*protolists));
    // Count backwards through cats to get newest categories first
    int mcount = 0;
    int propcount = 0;
    int protocount = 0;
    int i = cats->count; // 获取宿主类分类的总数
    bool fromBundle = NO;
    while (i--) { // 这里是倒序遍历，最先访问最后编译的分类
        // 获取一个分类
        auto &entry = cats->list[i];
        // 获取该分类的方法列表
        method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
        if (mlist) {
            // 最后编译的分类最先添加到分类数组中
            mlists[mcount++] = mlist;
            fromBundle |= entry.hi->isBundle();
        }
        // 属性列表添加规则 同方法列表添加规则
        property_list_t *proplist = entry.cat->propertiesForMeta(isMeta, entry.hi);
        if (proplist) {
            proplists[propcount++] = proplist;
        }
        // 协议列表添加规则 同方法列表添加规则
        protocol_list_t *protolist = entry.cat->protocols;
        if (protolist) {
            protolists[protocount++] = protolist;
        }
    }

    // 获取宿主类当中的 rw 数据，其中包含宿主类的方法列表信息
    auto rw = cls->data();

    // 主要是针对 分类中有关于内存管理相关方法情况下的一些特殊处理
    prepareMethodLists(cls, mlists, mcount, NO, fromBundle);

    /*
     rw 代表类
     methods 代表类的方法列表
     attachLists 方法的含义是 将含有 mcount 个元素的 mlists 拼接到 rw 的 methods 上
     */
    rw->methods.attachLists(mlists, mcount);
    free(mlists);
    if (flush_caches && mcount > 0)
        flushCaches(cls);
    rw->properties.attachLists(proplists, propcount);
    free(proplists);
    rw->protocols.attachLists(protolists, protocount);
    free(protolists);
}

void attachLists(List *const *addedLists, uint32_t addedCount) {
    if (addedCount == 0)
        return;
    /*
     addedLists 传递过来的二维数组
     [[method_t,method_t,...],[method_t],[method_t,method_t,method_t],...]
     分类中的方法列表 A B C
     addedCount = 3
     */

    addedCount * sizeof(array()->lists[0]));
    if (hasArray()) {
        // many lists -> many lists
        // 列表中原有元素总数 oldCount = 2
        uint32_t oldCount = array()->count;
        
        // 拼接之后的元素总数
        uint32_t newCount = oldCount + addedCount;
        
        // 根据新总数重新分配内存
        setArray((array_t *)realloc(array(), array_t::byteSize(newCount)));
        
        // 重新设置元素总数
        array()->count = newCount;
        
        /*
         内存移动
         [[], [], [], [原有的第一个元素], [原有的第二个元素]]
         */
        memmove(array()->lists + addedCount, array()->lists, oldCount * sizeof(array()->lists[0]));

        /*
         内存拷贝
         这也是分类方法会"覆盖"宿主类的方法的原因
         */
        memcpy(array()->lists, addedLists, addedCount * sizeof(array()->lists[0]));
        
    } else if (!list && addedCount == 1) {
        // 0 lists -> 1 list
        list = addedLists[0];
    } else {
        // 1 list -> many lists
        List *oldList = list;
        uint32_t oldCount = oldList ? 1 : 0;
        uint32_t newCount = oldCount + addedCount;
        setArray((array_t *)malloc(array_t::byteSize(newCount)));
        array()->count = newCount;
        if (oldList) {
            array()->lists[addedCount] = oldList;
        }
        memcpy(array()->lists, addedLists,
    }
}

```

### （五）通过 关联对象 为 Category 添加实例变量/关联对象的本质

在Category新增property的get和set方法里使用如下关联对象方法：

```
id objc_getAssociatedObject(id object,const void *key)

void objc_setAssociatedObject(id object,const void *key,id value,objc_AssociationPolicy policy)

void objc_removeAssociatedObjects(id object)
```

通过 key 获取 value，然后是以 policy（weak\strong等）的形式绑定到宿主对象 object上。

#### 1. 我们通过关联对象为Category添加成员变量，那么这个成员变量cache在哪里呢？

关联对象由 `AssociationsManager` 管理并在 `AssociationsHashMap`存储，所有对象的关联对象数据都在`同一个全局容器`中。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt0oryh5dyj30pk0bjt9r.jpg)

关联的流程是：

>> 0. 代码调用 `objc_setAssociatedObject` 添加关联对象会生成 `ObjcAssociation`数据结构
>> 1. 以 instanceProperty_name 作为 key ，ObjcAssociation 作为 value 生成 ObjectAssociationMap
>> 2. 以 宿主类的地址 为key，ObjectAssociationMap 为 value，完成宿主类和Category实例对象的绑定。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt0oxkq8kpj30t60cygmd.jpg)

#### 2. 源码分析

```
void _object_set_associative_reference(id object, void *key, id value, uintptr_t policy) {
    // retain the new value (if any) outside the lock. ObjcAssociation old_association(0, nil);
    // 根据策略 policy，对 value 进行加工
    id new_value = value ? acquireValue(value, policy) : nil;
    {
        // 关联对象管理类，C++实现的一个类
        AssociationsManager manager;
        // 获取其维护的一个 Hashmap，我们可以理解为是一个字典 // 是一个全局容器
        AssociationsHashMap &associations(manager.associations());
        disguised_ptr_t disguised_object = DISGUISE(object);
        if (new_value) {
            // break any existing association.
            // 根据对象指针查找对应的一个 ObjectAssociationMap 结构的 map
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
            if (i != associations.end()) {
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
    if (old_association.hasValue())
        ReleaseValue()(old_association);
}
```

## 二、扩展（Extension）

### (一)Extension的应用场景

- 声明私有属性
- 声明私有方法
- 声明私有成员变量

## 三、代理（delegate）

### （一）代理的特点

- 代理是一种软件设计模式
- 代理是一对一的方式传递的

### （二）解决delegate循环引用问题

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt0p0i73idj30pc05k3yo.jpg)

## 四、通知（NSNotification）

### 1. 通知的特点

- 使用`观察者模式`来实现`跨层传递消息`的效果
- 传递方式为 一对多

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt0p1pqp6jj30q708ejrv.jpg)

### 2. 如何实现通知

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt0p28wqedj30lg0b33z2.jpg)

## 五、KVO（Key-Value observing）

### （一）KVO的特点

- KVO 是OC对`观察者模式`的又一实现
- 苹果使用了 isa混写（isa-swizzling）技术 实现 KVO

### （二）如何通过 isa混写技术 实现 KVO ？

#### 1. 什么是isa指针

首先isa指针的全称，is a kind of 指针，顾名思义我们可以先理解为指向它所在类型的指针，如果一个类创建了一个实例，那么可以通过这个指针指向找到所在的类，下面打开objc.h文件

```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};
```

看的出每个objc_object对象都有一个指向Class类型的isa指针，再打开runtime.h文件

```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif
} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */
```

这里有Class类型的isa指针还有super_class，也可以看出isa指针并不是指向父类指针，这个结构体里面的内容很直观，isa指针、指向父类指针、名称、版本、信息、变量大小、变量列表、方法列表等等，每个objc_object可以通过isa指针找到它的类，并找到想要实现的方法或遵循的协议，由于objc_class也有isa指针，所以objc_class也是一个对象，称为“类对象”，它的isa指针指向他的元类(Meta-Class)，这样一来就很清晰了，每个对象通过isa指针向类中查找信息，类对象通过isa指针向元类查找信息，每个实例对象或类对象根据super_class指针都可以找到它们的父类，至此整个继承传递结构出来了

(isa + superClass) 完成了类对象的实例化与串联，

isa：是一个Class 类型的指针. 每个实例对象有个isa的指针,他指向对象的类，而Class里也有个isa的指针, 指向meteClass(元类)。元类保存了类方法的列表。当类方法被调用时，先会从本身查找类方法的实现，如果没有，元类会向他父类查找该方法。同时注意的是：元类（meteClass）也是类，它也是对象。元类也有isa指针,它的isa指针最终指向的是一个根元类(root meteClass).根元类的isa指针指向本身，这样形成了一个封闭的内循环。

super_class：父类，如果该类已经是最顶层的根类,那么它为NULL。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1aya8g09j30gm0hdwfh.jpg)

#### 2. 如何通过 isa混写 实现 KVO ？

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1b0zu8odj30rf0dqt9p.jpg)

KVO 流程：

>> 0. 调用 addObserveForKeyPath A.property
>> 1. 系统runtime创建 NSKVONotifying_A 一个新的类，同时将原先指向 A 类的isa指针指向新创建的类
>> 2. 在新创建的 NSKVONotifying_A 中重写要监听的 property 的 setter 方法

```
KVO 两个关键方法:

// 添加监听
- (void)addObserver:(NSObject *)observer
         forKeyPath:(NSString *)keyPath
            options:(NSKeyValueObservingOptions)options
            context:(nullable void *)context {
}

// 变更回调
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary<NSKeyValueChangeKey,id> *)change
                       context:(void *)context {
}

// 添加 KVO 必须在合适的时机移除！不然会crash
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;


```

NSKVONotifying_A 是 A 的一个子类，在 NSKVONotifying_A 中重写了 A 中相应 property 的 setter 方法。



### （三）调用流程与isa混写校验

#### 1. KVO 调用流程

##### （1）调用KVO

```
[obj addObserver:observer forKeyPath:@"value" options:NSKeyValueObservingOptionNew context:NULL];
```

##### （2）接收回调

```
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary<NSKeyValueChangeKey, id> *)change
                       context:(void *)context {
}
```

#### 2. isa混写校验

调用`po object_getClassName(obj)`，可以发现在添加 KVO 前后，obj的类型发生了改变。

KVO 之前：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1b804dwpj30r409hmyb.jpg)

KVO 之后：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1b8cjynfj30re0byjsr.jpg)

Tips: 如果对一个类添加多个 KVO，会进行多次 isa混写 吗？

不会的，只会多创建出一个 NSKVONotifying_object。 

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1b9chghsj30qv0ebjt2.jpg)


## 六、KVC（Key-Value Coding 键值编码）

### （一）KVC 的特点

KVC 是苹果提供的 可以直接「访问对象私有成员变量」和「修改私有成员变量值」的方法。

### （二）KVC 调用流程

#### 1. valueForKey 调用

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1bicgucrj30s50dpab1.jpg)

#### 2. setValue:ForKey: 调用

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1bltgskuj30tf0ctmy6.jpg)

### （三）关于 KVC 的问题

#### 1. KVC 是不是违背了 面向对象编程 的思想？

有没有违背 面向对象编程 的思想，我们首先要明白面向对象的 3 feature 和 5 principle 分别是什么、

3 feature：封装、继承、多态

5 principle：单一职责、开放封闭、Liskov替换、依赖倒置、接口隔离。

实际上 5 principle 对应的是 设计模式 ，KVC 实际上违背的是 面向对象 「3 feature」中的封装原则。

封装，就是将客观事物抽象为逻辑实体，实体的属性和功能相结合，形成一个有机的整体。并对实体的属性和功能实现进行访问控制，向信任的实体开放，对不信任的实体隐藏。通过开放的外部接口即可访问，无需知道功能如何实现。

比如一个类有私有属性，本身对外是没有暴露的，但外界却通过 KVC 直接修改了 私有属性，打破了「封装」的概念。

苹果提供了 KVC，也提供了 制衡 KVC 的方法，即：

```
+ (BOOL)accessInstanceVariablesDirectly {
    return NO;
}
```

如果`accessInstanceVariablesDirectly`方法的返回值为NO，也就意味着这个类不允许通过KVC来修改它的 私密属性 。注意了，是不允许修改私密属性，如果这个类本身已经把接口暴露出去了，那么通过 KVC 还是可以修改这个属性的。（原因可见上文 `setValueForKey` 调用流程图）


#### 2. 工程中什么时候会用到 KVC ？

项目中基本不会使用 KVC（微信工程里用到 KVC 的地方不多于10处）。

#### 3. KVC 改值会触发 KVO 吗？为什么

接着我们开始测试KVC、KVO，我们按如下流程来测试，测试的github代码如下：

[KVO、KVC测试Demo](https://github.com/BNineCoding/BNKVC_KVODemo)

##### （1）对私有property使用 KVC

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1c7y7aiqj30d306daa6.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1c7eorlej30q50dvta2.jpg)

#### （2）实例变量可以被 KVC 吗？

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1c8sash4j30ee05y0ss.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1c8zwwclj30qo0e2wfw.jpg)

可以

#### （3）对私有property 使用 KVC 会命中 property 的 setter 方法吗？

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1ca3t6iyj30r20f2q3t.jpg)

会命中

#### （4）对 私有实例变量 使用 KVC 会命中 setter 方法吗？

会命中。

私有实例变量没有可自动补全的 setter 方法，有的话也是自己暴露编写 getter/setter。

我发现使用 KVC，是会命中自己编写的 setter 方法的，所以即使没有自动补全 setter ，但系统在 runtime 阶段仍然承认自己编译的 setter 方法：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1eyghs4sj30ok0dmmy6.jpg)


#### （5）对 私有property 使用 KVO监听，然后使用 KVC 改值，KVO 会有回调吗？

有回调的。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1fdkcgg5j30x70j4416.jpg)

#### （6）对 私有成员变量 使用 KVO监听，然后使用 KVC 改值，KVO 会有回调吗？

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1ff5cylfj30yc0g7418.jpg)

## 七、属性关键字

### (一)关键字的类型

- 读写权限： readonly、readwrite
- 原子性：   nonatomic、atomic
- 引用计数： assign、copy、strong、weak

### （二）原子性问题（nonatomic、atomic）

思考:atomic 修改的属性关键字会产生什么效果?

atomic 修饰的属性实际上是可以保证赋值和获取(对成员属性的获取和赋值)是线程安全 的，这里 获取和赋值，并不代表 操作和访问。

比如通过 atomic 修饰数组，对数字进行赋值和获取是可以保证线程安全的，但如果对数组 进行操作，比如说给数组添加对象或者移除对象，是不在 atomic 负责范围内的。

所以说对 atomic 修饰的数组，进行添加对象和移除对象时，是没办法保证线程安全的「addObject: removeObject: 不安全」，只负责对数组的赋值和获取保证线程安全「getter/setter 安全」。

### （三）引用计数： assign、copy、strong、weak

#### 1. assign 的特点

- assign 修饰基本数据类型，如 int，BOOL 等; 
- 修饰对象类型时，不改变其引用计数;
- 会产生悬垂指针

assign 修饰的对象在被释放时，assign 指针仍然指向源对象地址，如果这时还通过原指针访问的话，可能会导致内存泄露或者程序异常。

问题：既然 assign 会产生悬垂指针，那么为什么 assign 修饰的 int、bool 在野指针释放时不会crash？

既然说到 crash，也思考一下为什么访问 野指针 会crash，因为我们调用了 野指针 的某些方法，因为指针已经野掉了，访问 野指针 的地址获取的并不是 我们预期的对象 ，那么如果调用了 这个 对象 的某些方法，那么就会crash。

但我们并不会调用 int\bool 的方法，只是直接访问，直接访问只是会导致数据异常，并不会crash。

#### 2. weak 的特点

- 不改变被修饰对象的引用计数(用 weak 多用于解决循环引用问题的)
- 所指对象在被释放之后会自动置为 nil。

问题： assgin 和 weak 有什么区别？

- weak 可以修饰对象，assign 既可以修饰对象、也可以修饰基本数据类型，但不建议使用 assign 修饰对象，如果指针野掉了，访问assign修饰对象的方法，就会crash
- assgin 修饰的对象在被释放后，assign 指针依旧指向源对象的地址;而weak 则会被置 为 nil。

### 3. copy 的特点

#### （1）浅拷贝 和 深拷贝

了解 copy 特点前我们先明确 浅拷贝 与 深拷贝。

##### a. 浅拷贝

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1gmorx8gj30bz07g3ye.jpg)

浅拷贝就是对内存地址的复制，让目标对象指针和源对象都指向同一片内存空间。

这样会导致「被拷贝对象」的「引用计数+1」，同时这种拷贝并没有「发生新的内存分配」

##### b. 深拷贝

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1go7dr0jj30s006ydfv.jpg)

深拷贝让目标对象指针和源对象指针指向两片内容相同的内存空间。

这样「不会增加被拷贝对象的引用计数」，但「产生了新的内存分配」

#### （2）copy 和 mutableCopy

copy 本意上是 浅拷贝，mutableCopy 是深拷贝。

但实际上效果是 浅拷贝 还是 深拷贝，还要根据拷贝的对象来区分，

如果对 mutableArray 进行拷贝，那么无论是 copy 还是 mutableCopy，实际都是 深拷贝。

如果对 array 进行拷贝，那么 copy 是 浅拷贝， mutableCopy 的结果是 深拷贝。

```
所以 copy ≠ 浅拷贝， mutableCopy == 深拷贝

但 copy == immutableCopy ，无论是 mutableArray 还是 array，

经过 copy 修饰后，都会变成 不可变对象。
```

问题：

`@property (copy) NSMutableArray *array`这种生命会导致什么问题？

答案：

假设 arrayA 是一个 NSMutableArray，

因为`array`生命为 copy ,执行 `array = arrayA` 之后，会将 `arrayA` 转成 `immutableArray` 然后赋值给 `array`，

如果接着再对 `array` 执行 NSMutableArray 的方法，就会以为找不到相应方法导致 crash。

所以一定不能随便将 对象 声明为 copy 。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)