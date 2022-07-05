---
title: isEqual、for安全遍历、json解包（小bug记）
date: 2021-08-01 23:40
tags: [小知识]
categories: [iOS]
---

# isEqual、for安全遍历、json解包（小bug记）

## 一、isEqual 踩坑

isEqual 其实没啥坑的，是我使用的时候发现了个使用的误区。

起因是在给同事提供接口判断一个对象的内容是否发生了变化，其实很简单，假设`BNObject`中有3个string：

```
@interface BNObject : NSObject

@property (nonatomic, copy) NSString *stringA;
@property (nonatomic, copy) NSString *stringB;
@property (nonatomic, copy) NSString *stringC;

@end
```

那么提供的`isEqual:`接口是：

```
- (BOOL)isEqual:(id)object {
    if (object == self) {
        return YES;
    } else if (![object isKindOfClass:[BNObject class]]) {
        return NO;
    } else {
        BNObject *objc = (BNObject *)object;
        BOOL stringAEqual = [self.stringA isEqualToString:objc.stringA];
        BOOL stringBEqual = [self.stringB isEqualToString:objc.stringB];
        BOOL stringCEqual = [self.stringC isEqualToString:objc.stringC];
        
        return stringAEqual && stringBEqual && stringCEqual;
    }
}
```

这个 `isEqual:`方法其实是有问题的，如果 `self.stringA` 和 `objc.stringA` 都是`nil`，那么应该判定为 `Equal`，但因为通过通过 nil 获取的 isEqualToString 默认是 NO，所以只采用`[A isEqualToString:B]`的判定方式，即使A和B都是nil，得到的结果也是NO。

正确的修正方式为：

```
- (BOOL)isEqual:(id)object {
    if (object == self) {
        return YES;
    } else if (![object isKindOfClass:[BNObject class]]) {
        return NO;
    } else {
        BNObject *objc = (BNObject *)object;
        BOOL stringAEqual = [BNObject stringIsEqual:self.stringA stringB:objc.stringA];
        BOOL stringBEqual = [BNObject stringIsEqual:self.stringB stringB:objc.stringB];
        BOOL stringCEqual = [BNObject stringIsEqual:self.stringC stringB:objc.stringC];
        
        return stringAEqual && stringBEqual && stringCEqual;
    }
}

// 判断两个字符串相等，都为nil也表示相等
+ (BOOL)stringIsEqual:(NSString *)stringA stringB:(NSString *)stringB {
    if (stringA.length <= 0 && stringB.length <= 0) {
        return YES;
    }
    return [stringA isEqualToString:stringB];
}
```

2021年10月30日更新:

使用 `yy_modelIsEqual`，可以不用对每个字段都写判等条件。

## 二、for 安全遍历踩坑

这实际上是已经很早之前的bug了，突然想起来也是记录一下。

背景是这样：我需要在遍历一个数组的时候，安全的删掉里面的元素，这是一个简单的面试题。

最初我的写法是这样的：

```
    NSMutableArray *array = [NSMutableArray arrayWithArray:@[@"1",@"2",@"3",@"4",@"5"]];
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isEqualToString:@"3"]) {
            [array removeObject:obj];
        }
    }];
    NSLog(@"%@",array);
```

实际上这种写法在每次都删除一个元素时，是正常的，上面这行代码输出的结果为：

```
(
    1,
    4,
    5
)
```

所以我的这种写法是没有安全问题的，只删除一个元素的情况下也是符合预期的。但如果是使用 enumerateObjectsUsingBlock 遍历时，删除了两个元素，就会出现index跳位的情况，甚至会导致crash：

```
    NSMutableArray *array = [NSMutableArray arrayWithArray:@[@"1",@"2",@"3",@"4",@"5"]];
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if ([obj isEqualToString:@"4"]) {
            [array removeObject:obj];
            [array removeObjectAtIndex:(idx + 1)];
        }
    }];
    NSLog(@"%@",array);
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1p6u3pe9j30qj0d8ju3.jpg)

所以终究来说，在 enumerateObjectsUsingBlock 遍历去删除元素，不是一个很优雅的做法，最稳妥的处理方式是：把需要删除的内容，筛选出来放到一个NSMutableArray中，然后再把这些需要删除的统一从原始数组中删除。

```
NSMutableArray *discardedItems = [NSMutableArray array];
SomeObjectClass *item;
for (item in originalArrayOfItems) {
    if ([item shouldBeDiscarded])
        [discardedItems addObject:item];
}
[originalArrayOfItems removeObjectsInArray:discardedItems];
```

## 三、json解包问题

这个问题我很早之前也遇到过，最近是一个实习生同学问到的：

有如下一个 json 字符串（层级 >= 2）：

```
{
	"name": "bninecoding",
	"like": {
		"sport": "basketball",
		"job": "code",
		"good": {
			"test": "hello"
		}
	}
}
```

客户端将 json 转成 NSDictionary 的方法是：

```
+ (NSDictionary *)dictionaryWithJsonString:(NSString *)jsonString {
    if (jsonString == nil) {
        return nil;
    }
    
    NSData *jsonData = [jsonString dataUsingEncoding:NSUTF8StringEncoding];
    NSError *err;
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData
                                                        options:NSJSONReadingMutableContainers
                                                          error:&err];
    
    if(err) {
        NSLog(@"json解析失败：%@",err);
        return nil;
    }
    
    return dic;
}
```

将 json 转成 NSDictionary 之后可以看到 dictionary 是会按 json 格式递归获取数据的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt1pgrfur1j30cq08aaae.jpg)

但实习生同学问到case是，它使用同样的方式去反解 json ，发现 json 无法递归生成 NSDictionary，建议 yy_model 去解 json串。

当时专家会诊得到的结论是通过 `NSJSONSerialization JSONObjectWithData:options:error:`不能递归去解json串。

其实得到的结论是错的，上面已经验证了，是可以递归解全json串的，最后看了实习生同学改的代码，发现是因为接口传过来的json格式错误导致的（预期是 NSString，但接口传成了 Dictionary）。

当然了，解 json 串，我还是建议使用 yy_model ，更清晰明了。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)