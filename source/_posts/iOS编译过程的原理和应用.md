---
title: iOS编译过程的原理和应用
date: 2021-08-09 00:57
tags: [编译原理]
categories: [计算机原理]
---

# iOS编译过程的原理和应用

阅读本文你将收获：

- iOS编译流程
- 明确 __text、__data 的含义
- iOS dSYM 的作用和生成方式
- 明白iOS crash堆栈符号化解析流程

source：

[iOS编译过程的原理和应用](https://github.com/LeoMobileDeveloper/Blogs/blob/master/iOS/iOS%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B%E7%9A%84%E5%8E%9F%E7%90%86%E5%92%8C%E5%BA%94%E7%94%A8.md)

[iOS 如何调试第三方统计到的崩溃报告](https://blog.csdn.net/hello_hwc/article/details/50036323)

[iOS Link Map File 文件说明](https://juejin.cn/post/6844904168096792583)

## 前言
一般可以将编程语言分为两种，[编译语言](https://zh.wikipedia.org/wiki/%E7%B7%A8%E8%AD%AF%E8%AA%9E%E8%A8%80)和[直译式语言](https://en.wikipedia.org/wiki/Interpreted_language)。

像C++,Objective C都是编译语言。编译语言在执行的时候，必须先通过编译器生成机器码，机器码可以直接在CPU上执行，所以执行效率较高。

像JavaScript,Python都是直译式语言。直译式语言不需要经过编译的过程，而是在执行的时候通过一个中间的解释器将代码解释为CPU可以执行的代码。所以，较编译语言来说，直译式语言效率低一些，但是编写的更灵活，也就是为啥JS大法好。

iOS开发目前的常用语言是：Objective和Swift。二者都是编译语言，换句话说都是需要编译才能执行的。二者的编译都是依赖于Clang + LLVM. 篇幅限制，本文只关注Objective C，因为原理上大同小异。

可能会有同学想问，我不懂编译的过程，写代码也没问题啊？这点我是不否定的。但是，充分理解了编译的过程，会对你的开发大有帮助。本文的最后，会以以下几个例子，来讲解如何合理利用XCode和编译

- `__attribute__`
- Clang警告处理
- 预处理
- 插入编译期脚本
- 提高项目编译速度

对于不想看我啰里八嗦讲一大堆原理的同学，可以直接跳到本文的最后一个章节。

----------
## 一、iOS编译

Objective C采用Clang(swift采用[swiftc](https://swift.org/compiler-stdlib/#compiler-architecture))作为编译器前端，LLVM作为编译器后端。

简单的编译过程如图

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9gqwzie4j31gq0dwdgc.jpg)

### （一）编译器前端

> 编译器前端的任务是进行：词法分析，语法分析，语义分析，生成中间代码(intermediate representation )。在这个过程中，会进行类型检查，如果发现错误或者警告会标注出来在哪一行。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9gro7znvj30zk0u040b.jpg)

### （二）编译器后端
> 编译器后端会进行机器无关的代码优化，生成机器语言，并且进行机器相关的代码优化。iOS的编译过程，后端的处理如下

- **LVVM优化器会进行BitCode的生成，链接期优化等等**。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9gsft959j315m0oot9v.jpg)

- **LLVM机器码生成器会针对不同的架构，比如arm64等生成不同的机器码**。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9gsthmuij312l0u03zy.jpg)


----------
## 二、执行一次XCode build的流程
当你在XCode中，选择build的时候（快捷键command+B），会执行如下过程

- 编译信息写入辅助文件，创建编译后的文件架构(name.app)
- 处理文件打包信息，例如在debug环境下
```
Entitlements:
{
    "application-identifier" = "app的bundleid";
    "aps-environment" = development;
}
```
- 执行CocoaPod编译前脚本
	- 例如对于使用CocoaPod的工程会执行`CheckPods Manifest.lock`
- 编译各个.m文件，使用`CompileC`和`clang`命令。
```
CompileC ClassName.o ClassName.m normal x86_64 objective-c com.apple.compilers.llvm.clang.1_0.compiler
export LANG=en_US.US-ASCII
export PATH="..."
clang -x objective-c -arch x86_64 -fmessage-length=0 -fobjc-arc... -Wno-missing-field-initializers ... -DDEBUG=1 ... -isysroot iPhoneSimulator10.1.sdk -fasm-blocks ... -I 上文提到的文件 -F 所需要的Framework  -iquote 所需要的Framework  ... -c ClassName.c -o ClassName.o
```
通过这个编译的命令，我们可以看到

```
clang是实际的编译命令
-x 		objective-c 指定了编译的语言
-arch 	x86_64制定了编译的架构，类似还有arm7等
-fobjc-arc 一些列-f开头的，指定了采用arc等信息。这个也就是为什么你可以对单独的一个.m文件采用非ARC编程。
-Wno-missing-field-initializers 一系列以-W开头的，指的是编译的警告选项，通过这些你可以定制化编译选项
-DDEBUG=1 一些列-D开头的，指的是预编译宏，通过这些宏可以实现条件编译
-iPhoneSimulator10.1.sdk 制定了编译采用的iOS SDK版本
-I 把编译信息写入指定的辅助文件
-F 链接所需要的Framework
-c ClassName.c 编译文件
-o ClassName.o 编译产物
```

- 链接需要的Framework，例如`Foundation.framework`,`AFNetworking.framework`,`ALiPay.fframework`
- 编译xib文件
- 拷贝xib，图片等资源文件到结果目录
- 编译ImageAssets
- 处理info.plist
- 执行CocoaPod脚本
- 拷贝Swift标准库
- 创建.app文件和对其签名

----------

## 三、IPA包的内容
例如，我们通过iTunes Store下载微信，然后获得ipa安装包，然后实际看看其安装包的内容。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9guja7w1j314g0owabc.jpg)

 - 右键ipa，重命名为`.zip`
 - 双击zip文件，解压缩后会得到一个文件夹。所以，ipa包就是一个普通的压缩包。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9guyea0gj30y608uwet.jpg)
 - 右键图中的`WeChat`，选择显示包内容，然后就能够看到实际的ipa包内容了。

------
## 四、二进制文件的内容

通过XCode的Link Map File，我们可以窥探二进制文件中布局。
在XCode -> Build Settings -> 搜索map -> 开启Write Link Map File

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9gvsjxifj31aw0c8gnm.jpg)

开启后，在编译，我们可以在对应的Debug/Release目录下看到对应的link map的text文件。
默认的目录在
```
~/Library/Developer/Xcode/DerivedData/<TARGET-NAME>-对应ID/Build/Intermediates/<TARGET-NAME>.build/Debug-iphoneos/<TARGET-NAME>.build/
```
例如，我的TargetName是`EPlusPan4Phone`，目录如下
```
/Users/huangwenchen/Library/Developer/Xcode/DerivedData/EPlusPan4Phone-eznmxzawtlhpmadnbyhafnpqpizo/Build/Intermediates/EPlusPan4Phone.build/Debug-iphonesimulator/EPlusPan4Phone.build
```

Tips：
Derived Data是一个文件夹，它默认情况下位于： ~/Library/Developer/Xcode/DerivedData 。 它是Xcode存储各种中间构建结果、生成索引等的位置。

### （一）iOS Link Map File 文件说明

#### 1. Link Map File 是什么

Link Map File 直译为 链接映射文件，是 Xcode 生成可执行文件时一起生成的文本，用于记录链接相关信息。

- 可执行文件的路径
- CPU架构
- .o目标路径
- 方法符号

#### 2. Link Map File 有什么用

- 查看代码加载顺序
- 理解内存分段分区
- Crash 时通过 Symbols 定位源码的机制
- 分析可执行文件中类或库体积，优化包体积

#### 3. 生成 Link Map File

Xcode 在生成可执行文件的时候默认情况下不生成该文件。
在Xcode的配置中 Target -> Build Setting -> Linking
将Write Link Map File设置为YES来生成Link Map File，运行代码即可生成Link Map File

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9hwupw3sj31b409eab7.jpg)

Linking下还可以看到生成文件的路径

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9hx3b3mfj31py0d60vi.jpg)

通过这个路径可以访问到

~/Developer/Xcode/DerivedData/项目/Build/Intermediates.noindex/项目.build/Debug-iphonesimulator/项目.build/项目-LinkMap-normal-x86_64.txt

还有一个简便方法，在Products下找到.app文件，返回上层后根据路径找到Link Map

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9hz9meb8j31c00es0um.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9hzep40tj31kk0hajtt.jpg)

#### 4. 查看 Link Map File

Link Map File主要分为3个部分

- 路径部分，展示生成的相关文件路径
- Section部分，展示相关地址段
- Symbols部分，方法符号段

##### （1）路径部分

- Path是.app文件路径
- Object files是.o文件路径

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9i73d0yuj30ws0re12b.jpg)

#### （2）计算机系统知识

##### a. text 段

这部分区域的大小在程序运行前就已经确定，并且内存区域通常属于只读(某些架构也允许代码段为可写，即允许修改程序)。

代码段（code segment/text segment）通常是指用来存放程序执行代码的一块内存区域。

在代码段中，也有可能包含一些只读的常数变量，例如字符串常量等。 

##### b. data 段

数据段（data segment）通常是指用来存放程序中已初始化的全局变量的一块内存区域。

##### c. bss 段

bss段（bss segment）通常是指用来存放程序中未初始化的全局变量的一块内存区域。

bss是英文Block Started by Symbol的简称。

##### d. 堆（heap）

堆是用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。

当进程调用malloc等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）；

当利用free等函数释放内存时，被释放的内存从堆中被剔除（堆被缩减）。

##### e. 栈(stack)

栈又称堆栈，是用户存放程序临时创建的局部变量，

也就是说我们函数括弧“{}”中定义的变量（但不包括static声明的变量，static意味着在数据段中存放变量）。

除此以外，在函数被调用时，其参数也会被压入发起调用的进程栈中，并且待到调用结束后，函数的返回值也会被存放回栈中。

由于栈的先进先出(FIFO)特点，所以栈特别方便用来保存/恢复调用现场。

从这个意义上讲，我们可以把堆栈看成一个寄存、交换临时数据的内存区。 

#### （3）Section 部分

Mach-O 文件中的虚拟地址最终会映射到物理地址上。这些地址被分成不同的Segement： __TEXT段、__DATA段、__LINKEDIT段。

- __TEXT 包含 Mach header，被执行的代码和只读常量（如C 字符串），只读可执行（r-x）。
- __DATA 包含全局变量，静态变量等，可读写（rw-）。
- __LINKEDIT 包含了加载程序的元数据，比如函数的名称和地址，只读（r–）。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9ije6j63j30q60nywk9.jpg)

Segement 划分成了不同的 Section，不同的 Section 存储着不同的信息，下面是一些常用的 Section 的介绍。

##### a. __TEXT段中的 Section

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9il3qxalj30am0mf3zf.jpg)

##### b. __DATA段中的 Section

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9ilojsgcj30by0vr40h.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9im4rzk9j30bw04bt8o.jpg)

#### （4） Symbols 部分

- Address：方法代码的地址
- Size：方法占用的空间
- File：文件的编号
- Name：.o文件里面的方法符号

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9inyug2ij30wi0ccn0i.jpg)

#### （5）二进制重排

二进制重排 就是要重新排列这些 方法符号的顺序 ，中心思想就是把启动用到的代码挪到前面的位置加载！

### （二）解读 Link Map File 的内容

这个映射文件的主要包含以下部分：

#### 1. **Object files**

这个部分包括的内容
-  .o 文文件，也就是上文提到的.m文件编译后的结果。
- .a文件 
- 需要link的framework
 
 >  #！ Arch: x86_64
 >  #Object files:
>[0] linker synthesized
[1] /EPlusPan4Phone.build/EPlusPan4Phone.app.xcent
[2]/EPlusPan4Phone.build/Objects-normal/x86_64/ULWBigResponseButton.o
...
[1175]/UMSocial_Sdk_4.4/libUMSocial_Sdk_4.4.a(UMSocialJob.o)
[1188]/iPhoneSimulator10.1.sdk/System/Library/Frameworks//Foundation.framework/Foundation

这个区域的存储内容比较简单：前面是文件的编号，后面是文件的路径。<font color="red">文件的编号在后续会用到</font>

#### 2. **Sections** 

这个区域提供了各个段（Segment）和节（Section）在可执行文件中的位置和大小。这个区域完整的描述克可执行文件中的全部内容。

其中，段分为两种

 - __TEXT 代码段
 - __DATA 数据段 
 
例如，之前写的一个App，Sections区域如下，可以看到，代码段的

__text节的地址是0x1000021B0，大小是0x0077EBC3，而二者相加的下一个位置正好是__stubs的位置0x100780D74。


```
# Sections:
# 位置       大小        段       节
# Address	Size    	Segment	Section
0x1000021B0	0x0077EBC3	__TEXT	__text //代码
0x100780D74	0x00000FD8	__TEXT	__stubs
0x100781D4C	0x00001A50	__TEXT	__stub_helper
0x1007837A0	0x0001AD78	__TEXT	__const //常量
0x10079E518	0x00041EF7	__TEXT	__objc_methname //OC 方法名
0x1007E040F	0x00006E34	__TEXT	__objc_classname //OC 类名
0x1007E7243	0x00010498	__TEXT	__objc_methtype  //OC 方法类型
0x1007F76DC	0x0000E760	__TEXT	__gcc_except_tab 
0x100805E40	0x00071693	__TEXT	__cstring  //字符串
0x1008774D4	0x00004A9A	__TEXT	__ustring  
0x10087BF6E	0x00000149	__TEXT	__entitlements 
0x10087C0B8	0x0000D56C	__TEXT	__unwind_info 
0x100889628	0x000129C0	__TEXT	__eh_frame
0x10089C000	0x00000010	__DATA	__nl_symbol_ptr
0x10089C010	0x000012C8	__DATA	__got
0x10089D2D8	0x00001520	__DATA	__la_symbol_ptr
0x10089E7F8	0x00000038	__DATA	__mod_init_func
0x10089E840	0x0003E140	__DATA	__const //常量
0x1008DC980	0x0002D840	__DATA	__cfstring
0x10090A1C0	0x000022D8	__DATA	__objc_classlist // OC 方法列表
0x10090C498	0x00000010	__DATA	__objc_nlclslist 
0x10090C4A8	0x00000218	__DATA	__objc_catlist
0x10090C6C0	0x00000008	__DATA	__objc_nlcatlist
0x10090C6C8	0x00000510	__DATA	__objc_protolist // OC协议列表
0x10090CBD8	0x00000008	__DATA	__objc_imageinfo
0x10090CBE0	0x00129280	__DATA	__objc_const // OC 常量
0x100A35E60	0x00010908	__DATA	__objc_selrefs
0x100A46768	0x00000038	__DATA	__objc_protorefs 
0x100A467A0	0x000020E8	__DATA	__objc_classrefs 
0x100A48888	0x000019C0	__DATA	__objc_superrefs // OC 父类引用
0x100A4A248	0x0000A500	__DATA	__objc_ivar // OC ivar
0x100A54748	0x00015CC0	__DATA	__objc_data
0x100A6A420	0x00007A30	__DATA	__data
0x100A71E60	0x0005AF70	__DATA	__bss
0x100ACCDE0	0x00053A4C	__DATA	__common
```

#### 3. **Symbols**
Section部分将二进制文件进行了一级划分。而，Symbols对Section中的各个段进行了二级划分，
例如，对于`__TEXT	__text`,表示代码段中的代码内容。

```
0x1000021B0	0x0077EBC3	__TEXT	__text //代码
```
而对应的`Symbols`，起始地址也是`0x1000021B0 `。其中，文件编号和上文的编号对应

```
[2]/EPlusPan4Phone.build/Objects-normal/x86_64/ULWBigResponseButton.o
```

具体内容如下

```
# Symbols:
  地址     大小          文件编号    方法名
# Address	Size    	File       Name
0x1000021B0	0x00000109	[  2]     -[ULWBigResponseButton pointInside:withEvent:]
0x1000022C0	0x00000080	[  3]     -[ULWCategoryController liveAPI]
0x100002340	0x00000080	[  3]     -[ULWCategoryController categories]
....

```
到这里，我们知道OC的方法是如何存储的，我们再来看看ivar是如何存储的。
首先找到数据栈中`__DATA	__objc_ivar`

```
0x100A4A248	0x0000A500	__DATA	__objc_ivar
```
然后，搜索这个地址`0x100A4A248`，就能找到ivar的存储区域。

```
0x100A4A248	0x00000008	[  3] _OBJC_IVAR_$_ULWCategoryController._liveAPI
```

值得一提的是，对于String，会显式的存储到数据段中，例如,

```
0x1008065C2	0x00000029	[ 11] literal string: http://sns.whalecloud.com/sina2/callback

```

> 所以，若果你的加密Key以明文的形式写在文件里，是一件很危险的事情。

Tips:为什么危险了？难道通过ipa文件也能获取到 __text、__data 这些数据？

----------
## 五、dSYM 文件

全称：debug symbol

.dSYM文件是一个符号表文件, 这里面包含了一个16进制的保存函数地址映射信息的中转文件, 所有Debug的symbols都在这个文件中(包括文件名、函数名、行号等). 一般Xcode项目每次编译后, 都会产生一个新的.dSYM文件和.app文件, 这两者有一个共同的UUID.

我们在每次编译过后，都会生成一个dsym文件。dsym文件中，<font color="orange">存储了16进制的函数地址映射。</font>

在App实际执行的二进制文件中，是通过地址来调用方法的。在App crash的时候，第三方工具（Fabric,友盟等）会帮我们抓到崩溃的调用栈，调用栈里会包含crash地址的调用信息。然后，通过dSYM文件，我们就可以由地址映射到具体的函数位置。

XCode中，选择Window -> Organizer可以看到我们生成的archier文件

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9gxiaf87j31gy0lu769.jpg)

然后，

- 右键 -> 在finder中显示。
- 右键 -> 查看包内容。

关于如何用dsym文件来分析崩溃位置，可以查看我之前的一篇博客。

-  [iOS 如何调试第三方统计到的崩溃报告](http://blog.csdn.net/hello_hwc/article/details/50036323)

Tips：我明白客户端crash堆栈解析的流程了：

>> 上传 app store 前会编archier文件，这个文件存储了函数地址映射关系
>> 等上传完crash堆栈后，根据 archier 文件 就可以反解出符号表。

其实在 [使用 Time Profile 定位可复现掉帧bug](https://bninecoding.com/shi-yong-time-profile-ding-wei-ke-fu-xian-diao-zheng-bug.html)这个章节中我就已经接触到 dSYM 了，通过 dSYM 可以把卡顿的地方符号化处理。

----------
## 六、那些你想到和想不到的应用场景

### （一）iOS 如何调试第三方统计到的崩溃报告

原文：[iOS 如何调试第三方统计到的崩溃报告](https://blog.csdn.net/hello_hwc/article/details/50036323)

前言：App上线四个月了，陆陆续续也在友盟上收到一些崩溃报告。这里就写一篇博客，简单介绍下如何debug。

有一点要记住，不要指望着复现bug靠断点调试，有些bug，例如多线程引起的，很难复现

#### 1. 核心命令

`dwarfdump --lookup 0x0007434d  -arch arm7 uau1.3.0.app.dSYM`

#### 2. dSYM文件

XCode中
Window->Organizer->选择提交到App Store的Archies->右键，showin finder
右键－>显示包内容，在dsym文件件里的就是符号表文件。这里包含着debug信息

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9izer9ubj31e80cc75j.jpg)

或者window->Orgainzer，直接download dsym

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9izmmyugj30rw0t8dhs.jpg)

#### 3. 崩溃报告

以我收到的这个为例

```
-[NSNull length]: unrecognized selector sent to instance 0x198705e70
(null)
(
    0   CoreFoundation                      0x00000001862442f4 <redacted> + 160
    1   libobjc.A.dylib                     0x0000000197a680e4 objc_exception_throw + 60
    2   CoreFoundation                      0x000000018624b3a4 <redacted> + 0
    3   CoreFoundation                      0x0000000186248154 <redacted> + 928
    4   CoreFoundation                      0x000000018614accc _CF_forwarding_prep_0 + 92
    5   Foundation                          0x00000001871b2d98 <redacted> + 212
    6   Foundation                          0x00000001871b2c88 <redacted> + 200
    7   Foundation                          0x00000001871b3014 <redacted> + 52
    8   ?????????                           0x00000001000bf900 ????????? + 358656
    9   ?????????                           0x00000001000bf748 ????????? + 358216
    10  libdispatch.dylib                   0x00000001980b9994 <redacted> + 24
    11  libdispatch.dylib                   0x00000001980b9954 <redacted> + 16
    12  libdispatch.dylib                   0x00000001980be20c _dispatch_main_queue_callback_4CF + 1608
    13  CoreFoundation                      0x00000001861fb7f8 <redacted> + 12
    14  CoreFoundation                      0x00000001861f98a0 <redacted> + 1492
    15  CoreFoundation                      0x00000001861252d4 CFRunLoopRunSpecific + 396
    16  GraphicsServices                    0x000000018f93b6fc GSEventRunModal + 168
    17  UIKit                               0x000000018aceafac UIApplicationMain + 1488
    18  ?????????                           0x000000010007528c ????????? + 53900
    19  libdyld.dylib                       0x00000001980e6a08 <redacted> + 4
)

dSYM UUID: 916F22F5-F3B1-3709-BB7E-5B11267B5D1F
CPU Type: arm64
Slide Address: 0x0000000100000000
Binary Image: ???
Base Address: 0x0000000100068000
```

第一部分，看看UUID

`916F22F5-F3B1-3709-BB7E-5B11267B5D1F`

使用命令查看dsym文件的uuid

```
huangwenchendeMacBook-Pro:Dsym调试 huangwenchen$ dwarfdump --uuid uau1.3.0.app.dSYM/
UUID: EC087835-FA55-36F2-B4D9-430BF6C2BA69 (armv7) uau1.3.0.app.dSYM/Contents/Resources/DWARF/Name
UUID: 916F22F5-F3B1-3709-BB7E-5B11267B5D1F (arm64) uau1.3.0.app.dSYM/Contents/Resources/DWARF/Name
```

对照可以看到，uuid是一致的，那么这个dsym文件可用

#### 4. 对于 uuid 的理解

每个人手机的 uuid 都是不一样的吗？ 每个 app 的 uuid 是相同的吗？

UUID是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。

第3部分的背景是说每一个 app 的ipa包有一个唯一的 uuid，方便我们定位是哪个包。

#### 5. 找到?????部分

之所以要找到这部分，是因为crash log中call tree没办法识别的部分，往往就是自己代码的部分。

我的Crash Log中，分别是这三行

```
    8   ?????????                           0x00000001000bf900 ????????? + 358656
    9   ?????????                           0x00000001000bf748 ????????? + 358216
    18  ?????????                           0x000000010007528c ????????? + 53900
```

然后，用命令定位到代码crash处

`dwarfdump --lookup 0x000000010007528c  -arch arm64 uau1.3.0.app.dSYM`

看到Log

```
Looking up address: 0x000000010007528c in .debug_info... found!

0x000ada5c: Compile Unit: length = 0x00001139  version = 0x0002  abbr_offset = 0x00000000  addr_size = 0x08  (next CU at 0x000aeb99)

0x000ada67: TAG_compile_unit [99] *
             AT_producer( "Apple LLVM version 7.0.0 (clang-700.1.76)" )
             AT_language( DW_LANG_ObjC )
             AT_name( "/Users/huangwenchen/Desktop/nativeios/StampApp/StampApp/UAUOneStampMarketInfo.m" )
             AT_stmt_list( 0x0003647f )
             AT_comp_dir( "/Users/huangwenchen/Desktop/nativeios/StampApp" )
             AT_APPLE_optimized( 0x01 )
             AT_APPLE_major_runtime_vers( 0x02 )
             AT_low_pc( 0x0000000100074674 )
             AT_high_pc( 0x000000010007597c )

0x000adf2d:     TAG_subprogram [106] *
                 AT_low_pc( 0x0000000100075278 )
                 AT_high_pc( 0x0000000100075298 )
                 AT_frame_base( reg31 )
                 AT_object_pointer( {0x000adf50} )
                 AT_name( "-[UAUOneStampMarketInfo insitutionName]" )
                 AT_decl_file( "/Users/huangwenchen/Desktop/nativeios/StampApp/StampApp/UAUOneStampMarketInfo.m" )
                 AT_decl_line( 122 )
                 AT_prototyped( 0x01 )
                 AT_type( {0x000adcd1} ( NSString* ) )
                 AT_APPLE_optimized( 0x01 )
Line table dir : '/Users/huangwenchen/Desktop/nativeios/StampApp/StampApp'
Line table file: 'UAUOneStampMarketInfo.m' line 123, column 12 with start address 0x0000000100075288

Looking up address: 0x000000010007528c in .debug_frame... not found.
1
```

其中，这两行

```
AT_name( "-[UAUOneStampMarketInfo insitutionName]" )
                 AT_decl_file( "/Users/huangwenchen/Desktop/nativeios/StampApp/StampApp/UAUOneStampMarketInfo.m" )
```

基本上定定位到了UAUOneStampMarketInfo.m，中的insitutionName方法。
再结合

```
[NSNull length]:
这个错误，也就定位到了是因为后台返回了NSNull对象，自己没做检查
```

### （二）` __attribute__`
或多或少，你都会在第三方库或者iOS的头文件中，见到过__attribute__。
比如

```
__attribute__ ((warn_unused_result)) //如果没有使用返回值，编译的时候给出警告
```

> `__attribtue__` 是一个高级的的编译器指令，它允许开发者指定更更多的编译检查和一些高级的编译期优化。

分为三种：

> - 函数属性 （Function Attribute）
> - 类型属性  (Variable Attribute )
> - 变量属性  (Type Attribute )

语法结构

`__attribute__` 语法格式为：`__attribute__ ((attribute-list))`
放在声明分号“;”前面。

比如，在三方库中最常见的，声明一个属性或者方法在当前版本弃用了

```
@property (strong,nonatomic)CLASSNAME * property __deprecated;
```
这样的好处是：给开发者一个过渡的版本，让开发者知道这个属性被弃用了，应当使用最新的API，但是被__deprecated的属性仍然可以正常使用。如果直接弃用，会导致开发者在更新Pod的时候，代码无法运行了。

`__attribtue__`的使用场景很多，本文只列举iOS开发中常用的几个：

```
//弃用API，用作API更新
#define __deprecated	__attribute__((deprecated)) 

//带描述信息的弃用
#define __deprecated_msg(_msg) __attribute__((deprecated(_msg)))

//遇到__unavailable的变量/方法，编译器直接抛出Error
#define __unavailable	__attribute__((unavailable))

//告诉编译器，即使这个变量/方法 没被使用，也不要抛出警告
#define __unused	__attribute__((unused))

//和__unused相反
#define __used		__attribute__((used))

//如果不使用方法的返回值，进行警告
#define __result_use_check __attribute__((__warn_unused_result__))

//OC方法在Swift中不可用
#define __swift_unavailable(_msg)	__attribute__((__availability__(swift, unavailable, message=_msg)))

```
### （三）Clang警告处理
你一定还见过如下代码：

```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wundeclared-selector"
///代码
#pragma clang diagnostic pop
```
这段代码的作用是

1. 对当前编译环境进行压栈
2. 忽略`-Wundeclared-selector`（未声明的）Selector警告
3. 编译代码
4. 对编译环境进行出栈

通过clang diagnostic push/pop,你可以灵活的控制代码块的编译选项。

我在之前的一篇文章里，详细的介绍了XCode的警告相关内容。本文篇幅限制，就不详细讲解了。

-  [iOS 合理利用Clang警告来提高代码质量](http://blog.csdn.net/Hello_Hwc/article/details/46425503)

在这个链接，你可以找到所有的Clang warnings警告

- [fuckingclangwarnings](http://fuckingclangwarnings.com/)

### （四）预处理

所谓预处理，就是在编译之前的处理。预处理能够让你定义编译器变量，实现条件编译。
比如，这样的代码很常见

```
#ifdef DEBUG
//...
#else
//...
#endif
```
同样，我们同样也可以定义其他预处理变量,在XCode-选中Target-build settings中，搜索preprocess。然后点击图中蓝色的加号，可以分别为debug和release两种模式设置预处理宏。
比如我们加上：`TestServer`，表示在这个宏中的代码运行在测试服务器

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h01d6oaj31gc0u0q77.jpg)

然后，配合多个Target（右键Target，选择Duplicate），单独一个Target负责测试服务器。这样我们就不用每次切换测试服务器都要修改代码了。

```
#ifdef TESTMODE
//测试服务器相关的代码
#else
//生产服务器相关代码
#endif
```
### （五）插入脚本
通常，如果你使用CocoaPod来管理三方库，那么你的Build Phase是这样子的：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h099aibj31em0huq4c.jpg)

其中：[CP]开头的，就是CocoaPod插入的脚本。

- Check Pods Manifest.lock，用来检查cocoapod管理的三方库是否需要更新
- Embed Pods Framework，运行脚本来链接三方库的静态/动态库
- Copy Pods Resources，运行脚本来拷贝三方库的资源文件 

而这些配置信息都存储在这个文件(.xcodeproj)里

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h0iqddpj30n80u4dj5.jpg)

到这里，CocoaPod的原理也就大致搞清楚了，通过修改xcodeproject，然后配置编译期脚本，来保证三方库能够正确的编译连接。

同样，我们也可以插入自己的脚本，来做一些额外的事情。比如，每次进行archive的时候，我们都必须手动调整target的build版本，如果一不小心，就会忘记。这个过程，我们可以通过插入脚本自动化。

```
buildNumber=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "${PROJECT_DIR}/${INFOPLIST_FILE}")
buildNumber=$(($buildNumber + 1))
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "${PROJECT_DIR}/${INFOPLIST_FILE}"
```
这段脚本其实很简单，读取当前pist的build版本号,然后对其加一，重新写入。

使用起来也很简单：

- Xcode - 选中Target - 选中build phase 
- 选择添加Run Script Phase

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h0rga5gj31340em75s.jpg)

- 然后把这段脚本拷贝进去，并且勾选Run Script Only When installing，保证只有我们在安装到设备上的时候，才会执行这段脚本。重命名脚本的名字为Auto Increase build number

- 然后，拖动这个脚本的到Link Binary With Libraries下面

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h0z3t2ij31300ly0un.jpg)

### （六）脚本编译打包

脚本化编译打包对于CI（持续集成）来说，十分有用。iOS开发中，编译打包必备的两个命令是：

```
//编译成.app
xcodebuild  -workspace $projectName.xcworkspace -scheme $projectName  -configuration $buildConfig clean build SYMROOT=$buildAppToDir
//打包
xcrun -sdk iphoneos PackageApplication -v $appDir/$projectName.app -o $appDir/$ipaName.ipa

通过info命令，可以查看到详细的文档
info xcodebuild
```
 - [完整的脚本](https://github.com/LeoMobileDeveloper/Blogs/blob/master/DemoProjects/Scripts/autoIPA.sh)，使用的时候，需要拷贝到工程的根目录


### （七）提高项目编译速度
通常，当项目很大，源代码和三方库引入很多的时候，我们会发现编译的速度很慢。在了解了XCode的编译过程后，我们可以从以下角度来优化编译速度：
#### 1. 查看编译时间
我们需要一个途径，能够看到编译的时间，这样才能有个对比，知道我们的优化究竟有没有效果。
对于XCode 8，关闭XCode，终端输入以下指令

```
$ defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES
```

然后，重启XCode，然后编译，你会在这里看到编译时间。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h16w1zkj31ko08k0u8.jpg)


代码层面的优化

#### 2. **forward declaration**
所谓`forward declaration`，就是`@class CLASSNAME`，而不是`#import CLASSNAME.h`。这样，编译器能大大提高#import的替换速度。

#### 3. 对常用的工具类进行打包（Framework/.a）
打包成Framework或者静态库，这样编译的时候这部分代码就不需要重新编译了。

#### 4. 常用头文件放到预编译文件里
XCode的pch文件是预编译文件，这里的内容在执行XCode build之前就已经被预编译，并且引入到每一个.m文件里了。

编译器选项优化
#### 5. Debug模式下，不生成dsym文件
上文提到了，dysm文件里存储了调试信息，在Debug模式下，我们可以借助XCode和LLDB进行调试。所以，不需要生成额外的dsym文件来降低编译速度。

#### 6. Debug开启`Build Active Architecture Only` 
在XCode -> Build Settings ->  Build Active Architecture Only 改为YES。这样做，可以只编译当前的版本，比如arm7/arm64等等，记得只开启Debug模式。这个选项在高版本的XCode中自动开启了。

#### 7. Debug模式下，关闭编译器优化

编译器优化

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt9h1dq6e0j317q0d4mz0.jpg)

------



------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)