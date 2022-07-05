---
title: NSURL构造方法crash小记
date: 2021-12-28 12:27
tags: [NSURL,crash]
categories: [iOS]
---

# NSURL构造方法crash小记

我们经常乎使用到 NSURL 的各种构造方法，比如：`[NSURL fileURLWithPath:path]`

实际上 `NSURL`的构造方法，全部都是高危的，如果`path`传入的是nil，那么就会crash。

我之前遇到过几次，但因为感觉都是小问题，fix完之后就没有记在心上，

可昨天又遇到了因为`path`为nil导致crash的问题，同个问题犯了很多次，不可接受。

遂记录于此。

所有使用到的 `NSURL 构造方法`，都需要先判断传入的string是否为nil，加一个如下的约束：

```
	if(! path || path <= 0) {
		return nil;
	}

	NSURL *url = [NSURL fileURLWithPath:path];
```

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)