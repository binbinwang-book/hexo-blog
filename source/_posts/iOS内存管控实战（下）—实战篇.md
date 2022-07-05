---
title: iOS内存管控实战（下）—实战篇
date: 2022-06-05 22:34
tags: [iOS,内存]
categories: [iOS]
---

# iOS内存管控实战（下）—实战篇

因文章单篇过长，按照 原理、分析工具 和 实战 拆分成上、中、下三部分，点击阅读。
- [iOS内存管控实战（上）—原理篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-shang-yuan-li-pian.html)
- [iOS内存管控实战（中）-分析工具篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-zhong-fen-xi-gong-ju-pian.html)
- [iOS内存管控实战（下）—实战篇](https://bninecoding.com/ios-nei-cun-guan-kong-shi-zhan-xia-shi-zhan-pian.html)

## 三、内存优化实战

### （一）图片压缩优化

近期我着手处理了一个多张高清大图裁剪时爆内存的问题，经过我的测试，原有的图片压缩逻辑在选了N张大图后，内存会飙升到2G左右。。。 确实不爆就怪了。

我们先从原理上简单介绍一下iOS加载图片时的内存的消耗情况。

#### 1. iOS图像渲染原理

##### （1）图像渲染管线 （Image Rendering Pipeline)

从 MVC 架构的角度来说，UIImage 代表了 Model，UIImageView 代表了 View. 那么渲染的过程我们可以这样很简单的表示：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xnpa1g7yj20w80cuwev.jpg)

Model 负责加载数据，View 负责展示数据。

但实际上，渲染的流程还有一个很重要的步骤：解码（Decode）。

为了了解Decode，首先我们需要了解Buffer这个概念。

##### （2）缓冲区 （Buffers）

Buffer 在计算机科学中，通常被定义为一段连续的内存，作为某种元素的队列来使用。

下面让我们来了解几种不同类型的 Buffer。

##### Image Buffers 

Image Buffers 代表了图片（Image）在内存中的表示。每个元素代表一个像素点的颜色，Buffer 大小与图像大小成正比

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xnqihkpyj20zp0bt0ty.jpg)

:) 注意这里说的图像大小指的是：图像分辨率，不是图像的文件大小。例如：有一个 590KB 的图片，分辨率是 2048px * 1536px，它实际使用的内存不是 590KB，而是2048 * 1536 * 4（4 bytes per pixel)） = 12 MB ，这是很恐怖的，不过这也是我们可以着重优化的地方。

##### frame buffer

frame buffer 代表了一帧在内存中的表示。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xnqztochj21080ikgnj.jpg)

##### Data Buffers

Data Buffers 代表了图片文件（Image file）在内存中的表示。这是图片的元数据，不同格式的图片文件有不同的编码格式。Data Buffers不直接描述像素点。 因此，Decode这一流程的引入，正是为了将Data Buffers转换为真正代表像素点的Image Buffer

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xnre6aq8j21080bhwfx.jpg)

因此，图像渲染管线，实际上是像这样的：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xnta8y3tj21410r8n0h.jpg)

#### 2. 图片裁剪优化

项目中原来的图片裁剪使用的是UIKit提供的API：

```
extension UIImage {
    
    //UIKit
    func resizeUI(size: CGSize) -> UIImage? {
        
        let hasAlpha = false
        let scale: CGFloat = 0.0 // Automatically use scale factor of main screen
        
        /**
         创建一个图片类型的上下文。调用UIGraphicsBeginImageContextWithOptions函数就可获得用来处理图片的图形上下文。利用该上下文，你就可以在其上进行绘图，并生成图片
         
         size：表示所要创建的图片的尺寸
         opaque：表示这个图层是否完全透明，如果图形完全不用透明最好设置为YES以优化位图的存储，这样可以让图层在渲染的时候效率更高
         scale：指定生成图片的缩放因子，这个缩放因子与UIImage的scale属性所指的含义是一致的。传入0则表示让图片的缩放因子根据屏幕的分辨率而变化，所以我们得到的图片不管是在单分辨率还是视网膜屏上看起来都会很好
         */
        UIGraphicsBeginImageContextWithOptions(size, !hasAlpha, scale)
        self.draw(in: CGRect(origin: .zero, size: size))
        
        let resizedImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return resizedImage!
    }
}
```

UIKit处理大分辨率图片时，往往容易出现OOM，原因是-[UIImage drawInRect:]在绘制时，先解码图片，再生成原始分辨率大小的bitmap，这是很耗内存的。解决方法是使用更低层的ImageIO接口，避免中间bitmap产生。

苹果官方在[Performance Best Practices](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_performance/ci_performance.html#//apple_ref/doc/uid/TP30001185-CH10-SW1)也给出了相关的建议：

**Use Core Graphics or Image I/O functions to crop or downsample, such as the functions CGImageCreateWithImageInRect or CGImageSourceCreateThumbnailAtIndex.**

所以我们可以将图片裁剪的API接口替换成 ImageIO ：

```
extension UIImage {
    
    //ImageIO
    func resizeIO(size:CGSize) -> UIImage? {
        
        guard let data = UIImagePNGRepresentation(self) else { return nil }
        
        let maxPixelSize = max(size.width, size.height)
        
        //let imageSource = CGImageSourceCreateWithURL(url, nil)
        guard let imageSource = CGImageSourceCreateWithData(data as CFData, nil) else { return nil }
        
        //kCGImageSourceThumbnailMaxPixelSize为生成缩略图的大小。当设置为800，如果图片本身大于800*600，则生成后图片大小为800*600，如果源图片为700*500，则生成图片为800*500
        let options: [NSString: Any] = [
            kCGImageSourceThumbnailMaxPixelSize: maxPixelSize,
            kCGImageSourceCreateThumbnailFromImageAlways: true
        ]
        
        let resizedImage = CGImageSourceCreateImageAtIndex(imageSource, 0, options as CFDictionary).flatMap{
            UIImage(cgImage: $0)
        }
        return resizedImage
    }
}
```

#### 3. 图片压缩优化

在原项目代码中，有需求需要将图片使用 `UIImageJPEGRepresentation` 将图片压缩到目标文件大小，代码逻辑大致如下：

```
    while (imageData.length / 1024 > 1024 * maxSize && imageCompressRate > 0) {
        imageCompressRate -= 0.1;
        imageData = UIImageJPEGRepresentation(thumbImg, imageCompressRate);
    }
```

可以看到这里采用的是使用 UIImageJPEGRepresentation 进行循环压缩，假设我们选取的是60MB的图片，而目标是1MB，可能使用 UIImageJPEGRepresentation 将压缩因子从1压缩到0.1都无法达成目标，但这个过程却要调用10次 UIImageJPEGRepresentation 进行压缩。

那么在一个方法里循环调用 UIImageJPEGRepresentation 会不会对内存产生压力呢？ 我们用下面代码进行测试：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    UIImage *image = [UIImage imageNamed:@"test_image.jpeg"];
    for (int i = 0; i < 20; i ++) {
        NSData *data = UIImageJPEGRepresentation(image, 1.0);
    }
}
```

然后观察Memory，可以发现明显出现了内存抖动：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h2xopv0yxmj20gq0gcwes.jpg)

峰值最高时内存达到了60MB，也就是说使用 UIImageJPEGRepresentation 进行图片压缩时，压缩过程中所开辟的内存并不会主动释放掉，而是等整个方法跑完之后才会进行释放，解决这类问题正式我们 @autoreleasepool 关键词的强项，在官方给出的 autoreleasepool 介绍中有这么一段话：

```
If you write a loop that creates many temporary objects.You may use an autorelease pool block inside the loop to dispose of those objects before the next iteration. Using an autorelease pool block in the loop helps to reduce the maximum memory footprint of the application.
③ 如果你在循环中创建了大量的临时对象(autoreleased对象)，你应该在每次循环迭代间使用autorelease pool，这样能在下次循环前对对象进行销毁从而有效地降低系统内存使用峰值
```

那么我们将 @autoreleasepool 加入我们的优化方案中：

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        UIImage *image = [UIImage imageNamed:@"test_image.jpeg"];
        for (int i = 0; i < 20; i ++) {
            @autoreleasepool {
                NSData *data = UIImageJPEGRepresentation(image, 1.0);
            }
        }
    });
}
```

但我们神奇地发现！使用 autoreleasepool 并没有减轻我们的内存抖升问题，毛用都没有！(暂时还想不明白为什么一直没释放这块内存，打算今年WWDC有机会问一下apple工程师)

那怎么办？我们要优化我们检索到最佳图片的路径，不要使用压缩因子递减0.1的方法，而是采用更有效率的检索方式，这里可以采用两种方式：

- 二分查找法
- 反比例模型计算法

其中 [反比例模型计算法](https://bninecoding.com/ru-he-you-ya-di-ya-suo-yi-zhang-tu-pian.html)是我自己摸索的压缩方式，仅供参考。

#### 5. 小插曲

##### 1. JPEF压缩因子为1时，图像甚至会变大

[Why image size get increased after UIImagePNGRepresentation?](https://stackoverflow.com/questions/52116141/why-image-size-get-increased-after-uiimagepngrepresentation)

##### 2. 图片裁剪参数scale是什么含义

scale 本质是缩放因子。

##### 图像的尺寸

image.size并不是实际的像素，只是显示在屏幕的尺寸。

显示的尺寸 = 实际的像素 / 缩放比例

获取实际的像素

- 宽：CGImageGetWidth(image.CGImage)
- 高：CGImageGetHeight(image.CGImage)

- image.size.width = CGImageGetWidth(image.CGImage) / image.scale
- image.size.height = CGImageGetHeight(image.CGImage) / image.scale



## 参考文章

- [WWDC 2018：iOS 内存深入研究](https://juejin.cn/post/6844903621276991502#heading-9)
- [iOS调试Block引用对象无法被释放的一个小技巧](https://cloud.tencent.com/developer/article/1508382)
- [iOS之深入解析Memory内存](https://blog.csdn.net/Forever_wj/article/details/120578784)
- [MLeaksFinder：精准 iOS 内存泄露检测工具](https://wereadteam.github.io/2016/02/22/MLeaksFinder/)
- [OS X Mavericks 中的内存压缩技术到底有多强大？](https://www.zhihu.com/question/21775223)
- [FBRetainCycleDetector + MLeaksFinder 阅读](https://www.jianshu.com/p/76250de94b93)
- [获取OC对象的所有强引用对象](https://blog.jerrychu.top/2021/01/10/%E8%8E%B7%E5%8F%96OC%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%89%80%E6%9C%89%E5%BC%BA%E5%BC%95%E7%94%A8%E5%AF%B9%E8%B1%A1/)
- [WWDC2018 图像最佳实践](https://juejin.cn/post/6844903618429059086)

文章首发：[问我社区](http://www.wenwoha.com/)