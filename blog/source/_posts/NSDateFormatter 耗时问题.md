---
title: NSDateFormatter 耗时问题
date: 2021-11-20 20:52
tags: [iOS, NSDateFormatter]
categories: [iOS]
---

# NSDateFormatter 耗时问题

上周写一个内部Debug工具时，用到 `NSDateFormatter` 去解析 `Date`，同事指出每次解析`Date`都创建`NSDateFormatter`比较耗时，建议使用`NSDateFormatter`单例。

[点击查看本文使用demo github链接]()

原代码如下：

```
formatter  = [[NSDateFormatter alloc] init];
[formatter setDateFormat:@"yyyy-MM-dd"];
string = [formatter stringFromDate:date];
```

今天周末做了个简单的测试，主要是两个目的：

- 1. 测试每次都重建 NSDateFormatter 去解析 NSDate 是否耗时
- 2. 检测是哪个环节耗时

## 一、验证重建NSDateFormatter解析是否耗时

```
    NSInteger count = 1000;
    CFTimeInterval begin;
    CFTimeInterval end;
    NSDateFormatter *formatter;
    NSString *string;
    NSDate *date = [NSDate date];
    
    // 1. 测试每次都重建 NSDateFormatter 去解析 NSDate 是否耗时
    {
        // 1.1 每次解析时间都创建一个Formatter
        begin = CACurrentMediaTime();
        for (int i = 0; i < count; i++) {
            formatter  = [[NSDateFormatter alloc] init];
            [formatter setDateFormat:@"yyyy-MM-dd"];
            string = [formatter stringFromDate:date];
        }
        end = CACurrentMediaTime();
        printf("NSDateFormatter:               %8.2f ms\n", (end - begin) * 1000);
    }
    
    {
        // 1.2 只用一个Formatter去解析时间
        begin = CACurrentMediaTime();
        formatter  = [[NSDateFormatter alloc] init];
        for (int i = 0; i < count; i++) {
            [formatter setDateFormat:@"yyyy-MM-dd"];
            string = [formatter stringFromDate:date];
        }
        end = CACurrentMediaTime();
        printf("NSDateFormatter once:         %8.2f ms\n", (end - begin) * 1000);
    }
```

输出结果：

```
NSDateFormatter:                  51.27 ms
NSDateFormatter once:             1.68 ms
```

可以发现每次解析都创建`NSDateFormatter`确实非常耗时，那么接下来我们的问题是：

耗时的原因是什么呢？解析时我们做了三部分：

- 1. 初始化 NSDateFormatter
- 2. 给 Formatter 标识解析格式
- 3. Formatter 解析 Date

接下来我们来看哪个环节最耗时。

## 二、检测耗时环节

```
    NSInteger count = 1000;
    CFTimeInterval begin;
    CFTimeInterval end;
    NSDateFormatter *formatter;
    NSString *string;
    NSDate *date = [NSDate date];
    
    // 2. 检测是哪个环节耗时
    CFTimeInterval a;
    CFTimeInterval b;
    CFTimeInterval c;
    {
        for (int i = 0; i < count; i++) {
            begin = CACurrentMediaTime();
            // 创建 Formatter
            formatter  = [[NSDateFormatter alloc] init];
            end = CACurrentMediaTime();
            a += (end-begin);
            
            begin = CACurrentMediaTime();
            // 给 Formatter 标识解析格式
            [formatter setDateFormat:@"yyyy-MM-dd"];
            end = CACurrentMediaTime();
            b += (end-begin);
            
            begin = CACurrentMediaTime();
            // Formatter 解析 Date
            string = [formatter stringFromDate:date];
            end = CACurrentMediaTime();
            c += (end-begin);
        }
        
        printf("NSDateFormatter:alloc               %8.2f ms\n", a * 1000);
        printf("NSDateFormatter:setFormat           %8.2f ms\n", b * 1000);
        printf("NSDateFormatter:stringFromDate      %8.2f ms\n", c * 1000);
    }
```

输出结果是：

```
NSDateFormatter:alloc                   5.53 ms
NSDateFormatter:setFormat               0.28 ms
NSDateFormatter:stringFromDate         44.63 ms
```

可以发现，耗时排序是：

```
「3. Formatter 解析 Date」 > 「1. 初始化 NSDateFormatter」 > 「2. 给 Formatter 标识解析格式」
```

「3. Formatter 解析 Date」这个环节我们无法规避，但「1. 初始化 NSDateFormatter」可以通过全局单例的方式来规避。



------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)