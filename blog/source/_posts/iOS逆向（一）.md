---
title: iOS逆向（一）
date: 2021-08-03 01:51
tags: [逆向]
categories: [iOS]
---

# iOS逆向（一）

这期主要从逆向去掉广告事例对iOS逆向知识进行分享，旨在对逆向知识和现状做一个大致的介绍。  
阅读时间：8分钟左右。

- 逆向的意义：
  
  大部分公司对于开发的需求都是远大于逆向，不知道逆向对于开发的修炼也完全没有影响，那么这期为什么要分享逆向呢？
  说到逆向，我们脑中总是想到拿到一个app，通过逆向技术就可以拿到这个app的代码，即使不做逆向工程，学习逆向对于了解iOS底层原理，明白iOS开发过程中潜在的安全风险都是有诸多益处的。但是也一定要注意，天网恢恢疏而不漏，千万不能用逆向去做坏事。

- 逆向知识前奏  
	- 逆向开始的第一步是拥有一台越狱手机，PP助手可以实现越狱效果，需要注意的是越狱分为‘完美越狱’和‘不完美越狱’，不同版本的越狱情况可以在[PP助手官网](http://jailbreak.25pp.com/ios/)查看；越狱虽然听着像违规，但实际上越狱已经获得法律认可，是合法的；  

	- 既然没有源代码，如何修改app内的代码呢？  
可以通过Cycript完成代码修改，也可以通过theos进行hook操作；  

	- 拿到.ipa软件包如何进行源代码的分析呢？什么是Mach-O文件？  
app从开发到安装至手机经历着如下的过程：’项目工程’经过’编译、链接、签名’生成 .app文件，接着对 .app文件进行zip压缩就生成了 .ipa文件；  
  所以说我们拿到 .ipa文件之后首先要进行解压缩，然后会在解压缩后的文件发现一个没有后缀的文件，这个文件也就是iOS中的可执行文件，也就是我们逆向的主角。

- 逆向去广告流程分析  
	- 获得app ipa安装包  
获得ipa包的途径有多个，可以通过iTunes/PP助手（需越狱）等途径获取，比较简单。（注意高版本的iTunes取消了应用商店，iTunes途径需降级，Mojava版本暂不兼容降级版iTunes）

	- 脱壳（砸壳）  
		- 什么是脱壳，为什么要脱壳  
			- a.要脱壳就要先介绍一下加壳。  
所谓加壳，就是利用特殊的算法，对可执行文件的编码进行改变（比如压缩、加密），以达到保护程序代码的目的。加壳的普遍过程是在ipa上传到App Store后，App Store对可执行文件进行加壳。  
			- b.脱壳与之相反，就是将未加密的可执行文件还原出来。  
    		- c.加壳有两种主要的方式：硬加壳和动态脱壳。  
硬脱壳指的是执行解密算法得到可执行文件；  
动态脱壳指的是在程序运行过程中，从内存中将可执行文件导出。

	- ②怎样脱壳
    	- iOS中有很多好用的脱壳工具：  
    [Clutch](https://github.com/KJCracks/Clutch);  
    [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted/)


- Reveal/Cycript  
在完成需求的时候总是有这样的情景：这部分代码不是我写的，但是我需要’修改功能/修复bug/数据上报’，那么这时候通过UI组件进行方法代码的定位是经常要做的事情。但是在逆向工程中，我们没办法使用Xcode这样方便的IDE通过UI界面可视化找到组件类名再查看绑定的方法进行定位，那么这时候就要使用Reveal。  
[Reveal的安装与使用参考文章](https://jakciehoo.github.io/2016/09/12/2016-09-12-using-reveal-to-debug)
  
- 下面展示一张通过Reveal窥测app UI的截图：![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2z5nfy1ej316z0u0jwe.jpg)
	- Reveal是什么？    
Reveal是一种APP UI分析的工具，不仅限于自己的app，可以实现Xcode。  
[官网学习资料](https://revealapp.com/)
	- Cycript是什么？  
 Cycript能够挂钩正在运行的进程，能够在运行时修改应用的很多东西，功能十分强大，在逆向过程中充当着Xcode的角色，Cycript混合了OC和JS语法，使得iOS开发者可以很容易上手使用。  
 Cycript可以轻松实现如下功能：  
    	- 根据地址获取对象;  
    	- 可以在内存中找出属于这个类的对象；  
    	- 打印视图层次；  
    	- 加载frameworks;  
    	- 使用NSLog;  
    	- 使用CGGeometry functions;  
    	- 输出对象的属性;  
    	- 根据类获取方法;  
    	- 获取当前控制器;  
    	- 增加分类;  
    	- 创建Block;  
[具体学习可以参考](http://www.cycript.org/)

	- Reveal/Cycript在项目中如何使用  
  打开reveal，在reveal中找到广告对应的按钮，以及右下角对应的内存地址，这里假设地址为adrs；  
  这时去掉广告的方法有两种，通过上面对Cycript的介绍，我们很容易想到使用Cycript切换到终端，使用如下命令：  
-         cy# [#adrs removeFromSuperView]  
	- 这时可以发现页面上的广告不见了，但是Cycript的操作知识内存层次的修改，如果我们重新进入广告见面，会发现广告依旧存在，也即没有根治广告。  
![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2z6e3qf4j30u20owq5t.jpg)
![](https://tva1.sinaimg.cn/large/008i3skNgy1gt2z6lmjtdj311c0d00u1.jpg)

	- 虽然我们改不了可执行文件的源码，但是我们这时有一个想法，如果我对源码注入一个脚本，可以更改掉广告的initWithFrame方法，让它每次加载都return nil，那么就可以达到目的了，那怎么做呢？这时就需要引入theos - tweak的开发过程。

- 通过theos制作hook，编写Tweak.xm脚本
	- theos是什么？  
 谈到逆向，一个经常出现的名次就是hook，而theos就是可以实现hook的工具，它可以覆写某个方法、添加新的方法、添加变量，像个钩子一样钉在可执行文件上。

	- theos的使用小例  
 去掉某个标签，找到imageView重写其set方法
-       %hook CategorySelectorButton
 	      - (void)setImageView:(id)arg     //不能漏写了参数，漏写了就是另外一种方法了
   	    {

        }
        %end


- 这里只是介绍了逆向的几个主要步骤和工具，诸如Makefile环境变量的设置，终端编译，make package && make install 等细节并没有介绍，感兴趣的可以Google，交流。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)