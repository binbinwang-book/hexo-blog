---
title: Json 序列化
date: 2021-09-12 13:07
tags: [Json]
categories: [技术方案]
---

# Json 序列化

## 前言

即将周末了，今天聊点轻松的，也是工作中发现的一个有趣的知识点，这个知识点可以用下面这个问题概括：

如果让你把一张图片塞到Json里，你知道怎么做吗？

如果你很清楚步骤，这篇文章就可以不用看了。

## Convert to Json 允许的字段类型

先说下背景，今天做需求时需要将一个自定义的model序列化为json串，我使用了 `yy_modelToJSONString` 方法发现获取到的一直都是 `nil`，于是我Debug进到 YYModel 中查找原因，在下面这样一段代码发现了问题:

```
- (NSData *)yy_modelToJSONDataWithConfigTag:(NSString *_Nullable)configTag {
    id jsonObject = [self yy_modelToJSONObjectWithConfigTag:configTag];
    if (!jsonObject)
        return nil;
    if ([NSJSONSerialization isValidJSONObject:jsonObject]) {
        return [NSJSONSerialization dataWithJSONObject:jsonObject options:0 error:NULL];
    } else {
        return nil;
    }
}
```

主要问题就是 `[NSJSONSerialization isValidJSONObject:jsonObject]`，我发现是能够正常获取 `jsonObject` 的（NSDictionary结构），但在`isValidJSONObject`判断时，返回了`NO`，`isValidJSONObject`是系统方法，我们看它是怎么说明的：

```
/* Returns YES if the given object can be converted to JSON data, NO otherwise. The object must have the following properties:
    - Top level object is an NSArray or NSDictionary
    - All objects are NSString, NSNumber, NSArray, NSDictionary, or NSNull
    - All dictionary keys are NSStrings
    - NSNumbers are not NaN or infinity
 Other rules may apply. Calling this method or attempting a conversion are the definitive ways to tell if a given object can be converted to JSON data.
 */
+ (BOOL)isValidJSONObject:(id)obj;
```

说得很清楚，顶层结构需要是NSArray、NSDictionary，允许转换的value只能是NSString, NSNumber, NSArray, NSDictionary, or NSNull。

于是我查看我自己在转换的自定义Model，发现自定义Model中有一个字段是 `NSData`类型，如此一来便导致 Convert to Json失败。

下图用Demo进行验证：

![](https://tva1.sinaimg.cn/large/008i3skNgy1guc3cqd722j60yc0bbgn302.jpg)

当 `NSDictionary` 填入基本数据类型时，是可以Convert to Json的。

![](https://tva1.sinaimg.cn/large/008i3skNgy1guc3fzmhauj60p00fbwgb02.jpg)

当我们将一个 `UIImage` 装入了 `NSDictionary`，发现 Convert to json fail。

## 把图片存到Json

其实这个问题很简单，我们把 UIImage 转成 Base64 字符串存到Json中即可，可以理解为 base64 是对二进制的一种描述。

![](https://tva1.sinaimg.cn/large/008i3skNgy1guc3nqcweaj60yi0hf42202.jpg)

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)