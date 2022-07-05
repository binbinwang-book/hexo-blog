---
title: 该不该用PromiseKit，一个iOS开发者的自我反问
date: 2022-04-19 20:52
tags: [PromiseKit]
categories: [iOS]
---


# 该不该用PromiseKit，一个iOS开发者的自我反问

## 一. 你的异步开发代码真的优雅吗？

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h1ee9c76xsj23340jodh6.jpg)

在没接触到 PromiseKit 前，我们写异步请求，用标志位控制并发cgi的情况，我们手动管理异步事件并不是一件麻烦事。但当真正看到 PromiseKit 要解决的问题后，才发觉 原来异步也可以如此优雅。

可以这么说，当你写异步代码时发现自己的代码风格有点那么不对劲，那么这时候你就可以考虑使用 PromiseKit 了，下面先举两个例子。

### （一）异步回调地狱

相信我们都写过类似这样的逻辑

```
    // 先注册，注册成功之后去登录，登录成功之后刷新状态
    [self signUpWithUserName:uid pwd:pwd resultBlock:^(BOOL success, NSString *message) {
        if (success) {
            [self signInWithUserName:uid pwd:pwd resultBlock:^(BOOL success, NSString *message) {
                if (success) {
                    NSLog(@"登录成功,uid:%@",uid);
                    [self asyncFetchAuthorInfoSucBloc:^(id data) {
                        NSLog(@"刷新状态成功");
                    }];
                }else{
                    NSLog(@"注册成功")
                }
            }];
        }else{
            NSLog(@"注册失败");
        }
    }];
```

可能我们写多了异步回调的代码，从逻辑上解释这个代码写的完全没有问题，而且相当好理解，但有没有发现这种代码属于 地狱回调 的代码风格，如果你写了 N个异步回调，你的代码风格会变成：

```
if xxx
	if yyy
	 	if zzz
	 		 if kkk
	 		 	if lll 
	 		 		if qqq
```

好，这里是一个例子，接下来我们看第二个例子。

### （二）控制并发异步回包逻辑

我们描述这么一个背景：

并发同时发起两个或多个异步请求，等到全部都完成后，再执行接下来的工作，代码里我们可能这么写：

```
int xxxFlag = 0;
[self fetchAAAAsyncInfoResultBlock:^(BOOL success) {
	if (success) {
		xxxFlag |= AAA;
		[self checkAsyncInfoLogic];
	}
}] 

[self fetchBBBAsyncInfoResultBlock:^(BOOL success) {
	if (success) {
		xxxFlag |= BBB;
		[self checkAsyncInfoLogic];
	}
}] 

- (void) checkAsyncInfoLogic {
	if ((xxxFlag & AAA) && (xxxFlag & BBB)) {
		异步均回包，进行下一步操作
	}
}
```

你可能会觉得这么写不是挺优雅的么，除了自己要稍微管理一个bit状态，我们觉得好是因为没见过更好的。如果我们有办法把 xxxFlag 标志位也省去了，那是不是更优雅呢？

示例说到这里，我们接下来首先介绍下 什么是 PromiseKit。

## 二. 什么是PromiseKit

Promise其实就是一个封装着异步操作的一个对象，它可以通过resolve以及reject来控制整个分支的流程。

下面是官方给出的一个定义：

>  A promise represents the future value of a task.

每个Promises都有一个状态，新建的每个Promises都处于pending状态，而后会转到resolve状态，resolve状态可以是fulfilled 或者 rejected ，如果是rejected 则会收到一个NSError，如果是fulfilled将会收到任何形式的对象。

```
                         |--> fulfilled
Promise init -> pending -|
                         |--> rejected
```



## 三. PromiseKit主要函数的使用方法

### （一）PromiseKit的创建

PromiseKit 的创建有两类方式：`Adapter`和`Resolve`：

Adapter 初始化：

```
+ (instancetype __nonnull)promiseWithAdapterBlock:(void (^ __nonnull)(PMKAdapter __nonnull adapter))block NS_REFINED_FOR_SWIFT;
+ (instancetype __nonnull)promiseWithIntegerAdapterBlock:(void (^ __nonnull)(PMKIntegerAdapter __nonnull adapter))block NS_REFINED_FOR_SWIFT;
+ (instancetype __nonnull)promiseWithBooleanAdapterBlock:(void (^ __nonnull)(PMKBooleanAdapter __nonnull adapter))block NS_REFINED_FOR_SWIFT;
```

Resolve 初始化：

```
+ (instancetype __nonnull)promiseWithResolverBlock:(void (^ __nonnull)(__nonnull PMKResolver))resolveBlock NS_REFINED_FOR_SWIFT;
```

那这两种初始化方法有什么区别呢？

我们查看 Adapter 源码可知：

```
+ (instancetype)promiseWithAdapterBlock:(void (^)(PMKAdapter))block {
    return [self promiseWithResolverBlock:^(PMKResolver resolve) {
        block(^(id value, id error){
            resolve(error ?: value);
        });
    }];
}
```

Adapter 的初始化方法底层调用了 Resolve 的方法，如果是 NSError 失败的场景，则将 NSError 传给 Resolve，反之则将对应的数据传给 Resolve。

其中 Adapter 传参对应的状态图如下：

![img](https://tbfungeek.github.io/2019/11/24/PromiseKit-%E7%9A%84%E4%BD%BF%E7%94%A8/00002.png)



### （二）then、catch、ensure关键词

#### 1. 获取Promise值

Promise值是通过then来获取的，PromiseKit目前有三个方法用于Promise值的获取，分别是then，thenInBackground，以及thenOn，分别在主线程，后台线程，指定线程获取Promise值。

```objective-c
- (AnyPromise * __nonnull (^ __nonnull)(id __nonnull))then NS_REFINED_FOR_SWIFT;
- (AnyPromise * __nonnull(^ __nonnull)(id __nonnull))thenInBackground NS_REFINED_FOR_SWIFT;
- (AnyPromise * __nonnull(^ __nonnull)(dispatch_queue_t __nonnull, id __nonnull))thenOn NS_REFINED_FOR_SWIFT;
```

#### 2. 捕获异常

在Promise reject 的时候，流程会走到catch分支，PromiseKit有两种catch方式来catch抛出的异常，一种在主线程，一种需要指定处理异常的线程。

```objective-c
- (AnyPromise * __nonnull(^ __nonnull)(id __nonnull))catch NS_REFINED_FOR_SWIFT;
- (AnyPromise * __nonnull(^ __nonnull)(id __nonnull))catchInBackground NS_REFINED_FOR_SWIFT;
- (AnyPromise * __nonnull(^ __nonnull)(dispatch_queue_t __nonnull, id __nonnull))catchOn NS_REFINED_FOR_SWIFT;
```

#### 3. 善后处理

在Promise 被 resolve的时候，不论结果是fullfill还是reject都会走到ensure这个分支。

```objective-c
- (AnyPromise * __nonnull(^ __nonnull)(dispatch_block_t __nonnull))ensure NS_REFINED_FOR_SWIFT;
- (AnyPromise * __nonnull(^ __nonnull)(dispatch_queue_t __nonnull, dispatch_block_t __nonnull))ensureOn NS_REFINED_FOR_SWIFT;
```



### （三）PMK系列接口

#### 1. PMKWHen 并发完成，单一失败

PMKWhen的参数是一个Promise数组,或者字典，它会等待所有的Promise执行结束，或者有一个Error发生，也就是说如果这些异步的Promise都没有Error的时候，会等到都Resolve后才执行then，并且then Block 传递回来的是各个Promise执行的结果，如果中途有Error发生就会中断，并且走到catch流程。下面是一个简单例子：

```objective-c
- (void)onCreate {
    [super onCreate];
    
    PMKWhen(@[[self promise_delay_1s],[self promise_delay_error],[self promise_delay_3s],[self promise_delay_8s]]).then(^(id value){
        NSLog(@"PMKWhen %@",value);
    }).catch(^(NSError *error){
        NSLog(@"PMKWhen %@",error);
    });
}

- (AnyPromise *)promise_delay_1s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_1s");
            resolver(@"PMKWhen promise_delay_1s");
        });
    }];
}

- (AnyPromise *)promise_delay_3s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_3s");
            resolver(@"PMKWhen promise_delay_3s");
        });
    }];
}

- (AnyPromise *)promise_delay_8s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(8 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_8s");
            resolver(@"PMKWhen promise_delay_3s");
        });
    }];
}

- (AnyPromise *)promise_delay_error {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(6 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_error");
            resolver([NSError errorWithDomain:@"test" code:100 userInfo:nil]);
        });
    }];
}
```

输出结果如下：

```
2019-11-04 14:19:46.896521+0800 IDLFundationTest[91339:11794858] PMKWhen promise_delay_1s
2019-11-04 14:19:48.994048+0800 IDLFundationTest[91339:11794858] PMKWhen promise_delay_3s
2019-11-04 14:19:52.477392+0800 IDLFundationTest[91339:11794858] PMKWhen promise_delay_error
2019-11-04 14:19:52.477859+0800 IDLFundationTest[91339:11794858] PMKWhen Error Domain=test Code=100 "(null)" UserInfo={NSUnderlyingError=0x600000c64450 {Error Domain=test Code=100 "(null)"}, PMKFailingPromiseIndexKey=0}
2019-11-04 14:19:54.694791+0800 IDLFundationTest[91339:11794858] PMKWhen promise_delay_8s
```

这里需要注意的是6s发生错误之后走catch分支，之后就不再走then分支了，但是error发生之后promise_delay_8s还会继续进行，并不会中止。



#### 2. PMKJoin 全部完成

这个和PMKWhen有类似的地方，就是接受的参数是字典或者数组，但是它会等待所有的Promise都解决后才走then或者catch分支，不像PMKWhen那样一旦有错误发生就catch，下面是一个例子可以对比下

```
- (void)onCreate {
    [super onCreate];
    
    PMKJoin(@[[self promise_delay_1s],[self promise_delay_error],[self promise_delay_3s],[self promise_delay_8s]]).then(^(id value){
        NSLog(@"PMKWhen %@",value);
    }).catch(^(NSError *error){
        NSLog(@"PMKWhen %@",error);
    });
}

- (AnyPromise *)promise_delay_1s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_1s");
            resolver(@"PMKWhen promise_delay_1s");
        });
    }];
}

- (AnyPromise *)promise_delay_3s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_3s");
            resolver(@"PMKWhen promise_delay_3s");
        });
    }];
}

- (AnyPromise *)promise_delay_8s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(8 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_8s");
            resolver(@"PMKWhen promise_delay_8s");
        });
    }];
}

- (AnyPromise *)promise_delay_error {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(6 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_error");
            resolver([NSError errorWithDomain:@"test" code:100 userInfo:nil]);
        });
    }];
}
```

下面是执行的结果：

```
2019-11-04 14:23:53.419624+0800 IDLFundationTest[94348:11805962] PMKWhen promise_delay_1s
2019-11-04 14:23:55.419830+0800 IDLFundationTest[94348:11805962] PMKWhen promise_delay_3s
2019-11-04 14:23:59.018698+0800 IDLFundationTest[94348:11805962] PMKWhen promise_delay_error
2019-11-04 14:24:01.218821+0800 IDLFundationTest[94348:11805962] PMKWhen promise_delay_8s
2019-11-04 14:24:01.221100+0800 IDLFundationTest[94348:11805962] PMKWhen Error Domain=PMKErrorDomain Code=10 "(null)" UserInfo={PMKJoinPromisesKey=(
    "AnyPromise(PMKWhen promise_delay_1s)",
    "AnyPromise(PMKWhen promise_delay_3s)",
    "AnyPromise(PMKWhen promise_delay_8s)"
```

可以看出catch是在全部Promise完成后执行的，并且在UserInfo中可以看出哪些是成功的Promise.

#### 3. PMKRace

PMKRace 会在第一个resolve(不论是fullfill还是reject)的时候执行then或者catch。

```
- (void)onCreate {
    [super onCreate];
    
    PMKRace(@[[self promise_delay_1s],[self promise_delay_error],[self promise_delay_3s],[self promise_delay_8s]]).then(^(id value){
        NSLog(@"PMKWhen %@",value);
    }).catch(^(NSError *error){
        NSLog(@"PMKWhen %@",error);
    });
}

- (AnyPromise *)promise_delay_1s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_1s");
            resolver(@"PMKWhen promise_delay_1s");
        });
    }];
}

- (AnyPromise *)promise_delay_3s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_3s");
            resolver(@"PMKWhen promise_delay_3s");
        });
    }];
}

- (AnyPromise *)promise_delay_8s {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(8 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_8s");
            resolver(@"PMKWhen promise_delay_8s");
        });
    }];
}

- (AnyPromise *)promise_delay_error {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver resolver) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(6 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"PMKWhen promise_delay_error");
            resolver([NSError errorWithDomain:@"test" code:100 userInfo:nil]);
        });
    }];
}
```

下面是执行结果：

```
2019-11-04 14:34:06.333085+0800 IDLFundationTest[2020:11831274] PMKWhen promise_delay_1s
2019-11-04 14:34:06.333917+0800 IDLFundationTest[2020:11831274] PMKWhen PMKWhen promise_delay_1s
2019-11-04 14:34:08.532678+0800 IDLFundationTest[2020:11831274] PMKWhen promise_delay_3s
2019-11-04 14:34:11.834737+0800 IDLFundationTest[2020:11831274] PMKWhen promise_delay_error
2019-11-04 14:34:14.034667+0800 IDLFundationTest[2020:11831274] PMKWhen promise_delay_8s
```

## 四. PromiseKit源码解析

## 五. PromiseKit应用场景和坑

说到这部分，我们可以针对第一部分的例子使用 PromiseKit 进行优化。

### （一）异步回调地狱优化

```

    [self promise_signUpWithUserName:uid pwd:pwd].then(^(id object){
        if ([object isKindOfClass:[NSArray class]]) {
            NSLog(@"test");
        }else{
            NSLog(@"helo");
        }
        return [self promise_signInWithUserName:uid pwd:pwd];
    }).then(^(NSString *msg) {
        NSLog(@"msg:%@",msg);
    }).catch(^(NSError *error) {
        NSLog(@"error:%@",error);
    });
    
    - (AnyPromise*)promise_signUpWithUserName:(NSString*)uid pwd:(NSString*)pwd {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver _Nonnull resolver) {
        // 网络请求 xxx
        if(YES) {
            resolver([NSArray arrayWithObject:@"1"]);
        }else {
            resolver([self errorWithMessage:@"注册失败！"]); // error自己定义
        }
    }];
}

- (AnyPromise*)promise_signInWithUserName:(NSString*)uid pwd:(NSString*)pwd {
    return [AnyPromise promiseWithResolverBlock:^(PMKResolver _Nonnull resolver) {
        // 网络请求 xxx
        if(NO) {
            resolver(@"登录成功");
        }else {
            resolver([self errorWithMessage:@"登录失败！"]); // error自己定义
        }
    }];
}

- (NSError*)errorWithMessage:(NSString*)msg {
    return [[NSError alloc] initWithDomain:NSNetServicesErrorDomain code:-101 userInfo:@{NSLocalizedDescriptionKey:msg}];
}
```



### （二）控制并发异步回包逻辑

控制并发异步回包逻辑，我们可以用上面已经介绍的 PMKWhen 进行处理，将多个异步行为打成一个包。

参考文章：

- [iOS 如何优雅的处理“回调地狱 Callback hell ”(一) —— 使用 PromiseKit](https://halfrost.com/ios_callback_hell_promisekit/)
- [Objective-C 之 PromiseKit入门](https://juejin.cn/post/6844903846427230221)
- [PromiseKit 的使用](https://tbfungeek.github.io/2019/11/24/PromiseKit-%E7%9A%84%E4%BD%BF%E7%94%A8/)