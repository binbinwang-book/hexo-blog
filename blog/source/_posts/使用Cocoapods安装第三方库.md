---
title: 使用Cocoapods安装第三方库
date: 2021-12-04 23:26
tags: [Cocoapods]
categories: [iOS]
---

# 使用Cocoapods安装第三方库

新建一个Xcode工程，使用终端cd到工程目录下

1. 创建Podfile文件：

>>> pod init

之后就可以在项目目录里看到一个Podfile文件

![](https://tva1.sinaimg.cn/large/008i3skNgy1gx27r3zdokj313208m0tr.jpg)

2. 打开Podfile文件：
>>> open Podfile

![](https://tva1.sinaimg.cn/large/008i3skNgy1gx27rgh6npj31840tgq5b.jpg)

添加：
>>> pod 'YYKit'

![](https://tva1.sinaimg.cn/large/008i3skNgy1gx28m16oajj30v40j275z.jpg)

保存后退出。

3. 开始下载：

>>> pod install

![](https://tva1.sinaimg.cn/large/008i3skNgy1gx28meq4tvj30x40h6jtn.jpg)

安装完毕。

4. 打开新生成的 project.xcworkspace

往后我们就需要使用这个新生成的 project.xcworkspace 文件来开发。因为原来的工程（SwiftDemo.xcodeproj）设置已经被更改了，如果我们直接打开原来的工程文件去编译就会报错。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gx28nd9nk4j30h803aglq.jpg)

重新打开工程后，我们导入:`#import <YYKit/YYKit.h>`已经可以编译成功了。


------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)