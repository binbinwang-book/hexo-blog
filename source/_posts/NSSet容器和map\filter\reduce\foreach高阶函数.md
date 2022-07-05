---
title: NSSet容器和map\filter\reduce\foreach高阶函数
date: 2021-07-04 02:34
tags: [NSSet,map,filter,reduece,foreach]
categories: [iOS]
---

# NSSet容器和map\filter\reduce\foreach高阶函数

今天提审下班比较晚，打算复习点轻松的知识。

## 一、NSSet 容器

### （一）无序

`NSSet` 表示集合，和`NSArray`有相似的地方，但 `NSSet`是无序的

### （二）唯一

`NSSet` 中存储的对象只能有唯一一个，不能重复

## 二、map\filter\reduce\foreach高阶函数

在`Swift`中，map filter reduce 等高阶函数的存在可以让我们更快的处理数据，虽然OC没有提供这些方法，我们也可以用代码实现同样的功能。

### （一）map

功能：处理数组中的每个元素，并返回一个新的结果数组

```
- (NSArray *)_map:(id(^)(id))handle {
    if (!handle || !self) return self;
    
    NSMutableArray *arr = NSMutableArray.array;
    for (id obj in self) {
        id new = handle(obj);
        [arr addObject:new];
    }
    return arr.copy;
}


/** Usages: */
NSArray *languages = @[@"objctive-c", @"java", @"swift",@"javascript", @"php"];
languages = [languages _map:^id _Nonnull(NSString *obj) {
  return [obj stringByAppendingString:@" !!"];
}];

NSLog(@"%@", languages);
/** Prints
(
    "objctive-c !!",
    "java !!",
    "swift !!",
    "javascript !!",
    "php !!"
)
*/
```

### （二）filter

功能：按照规则返回过滤之后的数组

```
- (NSArray *)_filter:(BOOL(^)(id))handle {
    if (!handle || !self) return self;
    
    NSMutableArray *arr = NSMutableArray.array;
    for (id obj in self) {
        if (handle(obj)) {
            [arr addObject:obj];
        }
    }
    return arr.copy;
}


/** Usages: */
NSArray *numbers = @[@1,@2,@5,@11,@6,@0,@3];
NSArray *oddArr = [numbers _filter:^BOOL(NSNumber *obj) {
  return (obj.intValue %2 != 0);
}];
    
NSLog(@"%@", oddArr);
/** Prints
(
    1,
    5,
    11,
    3
)
*/
```

### （三）reduce

功能：按照规则将组内元素一一合并，返回最终结果

比如把数组里的数字全部相乘然后输出，或者把文本全部拼接然后输出，用得比较少，但要知道这个用法。

```
- (id)_reduce:(id(^)(id, id))handle initial:(id)initial {
    if (!handle || !self || !initial) return self;
    if (self.count <1) return initial;
    
    id value = initial;
    for (id obj in self) {
        value = handle(value, obj);
    }
    return value;
}


/** Usages: */
NSArray *numbers = @[@3,@2,@10];
id result = [numbers _reduce:^id _Nonnull(NSNumber *obj1, NSNumber *obj2) {
  return @(obj1.intValue * obj2.intValue);
} initial:@1];
    
NSLog(@"%@", result);
// Prints  "60"

NSArray *words = @[@"hello", @"world", @"good", @"night"];
id result = [words _reduce:^id _Nonnull(NSString *obj1, NSString *obj2) {
  return [NSString stringWithFormat:@"%@%@", obj1, obj2];
} initial:@""];
    
NSLog(@"%@", result);
// Prints "helloworldgoodnight"
```

### （四）foreach

功能：对每个元素做操作

数组里已经有了 `enumerateObjectsUsingBlock:` 遍历方法，所以用到 foreach 的也很少。 

```
- (void)_forEach:(void(^)(id))handle {
    if (!handle || !self) return;
    
    for (id obj in self) {
        handle(obj);
    }
}


/** Usages: */
[subviews _forEach:^(UIView *view) {
  [view removeFromSuperview];
}];
```


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)