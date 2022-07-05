---
title: RGB、YUV、HSV和HSL区别和关联
date: 2022-03-20 21:30
tags: [RGB,YUV,HSV,HSL]
categories: [设计相关]
---

# RGB、YUV、HSV和HSL区别和关联

近期在做的一个需求和颜色转换有关系，所以本篇将开发过程中比较常见的 四种颜色 进行一番梳理。

## 一、RGB颜色空间

从我们最常见的RGB颜色出发，RGB分别对应着 Red（红）、Green（绿）、Blue（蓝），也就是我们平时所说的三原色，调整这三种颜色的比例，可以搭配出所有的色彩。

这时你可能就要问了，YUV、HSV、HSL也能描述所有色彩啊，为啥RGB是最常用的捏？

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0gn12dknoj20mn0a176b.jpg)

这就要回归到现实了，现实里显示器显像时，每一个像素点后面对应着 3个发光二极管，这3个二极管可以分别发出 红、绿、蓝 三种颜色，因此绝大部分人所能接触的颜色只与RGB有关系。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0gmvfqguzj20bf0a0aa2.jpg)

RGB（红绿蓝）是依据人眼识别的颜色定义出的空间，可表示大部分颜色。但在科学研究一般不采用RGB颜色空间，因为它的细节难以进行数字化的调整。它将色调，亮度，饱和度三个量放在一起表示，很难分开。它是最通用的面向**硬件**的彩色模型。该模型用于彩色监视器和一大类彩色视频摄像。

## 二、YUV颜色空间

YUV 多出现在音视频合成领域，音视频合成领域要求在表达同样内容时，争取占用更少的空间。同个视频，YUV空间要比RGB空间描绘省下来一半的空间消耗（YUV4:2:0）。

YUV（也称：YCbCr）：Y表示明亮度，UV的作用是描述影像色彩及饱和度。

主要的采样格式有 YUV4:2:0（最常用）、YUV4:2:2 和 YUV4:4:4 ，也就是说 RGB 主要用于屏幕图像的展示，而 YUV 多用于采集与编码。

YUV 和 RGB 相互转换的公式为：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0gn63r606j20ko07wgly.jpg)

## 三、HSV（HSB） 和 HSL

可以发现 RGB 主要为硬件显示器服务，YUV 主要为音视频编解码服务，这么说下来和色彩最亲密的 设计师 该用哪种颜色呢？ 

他们也有自己行业特别关注的颜色，主要使用 HSV 和 HSL。

### （一）为什么RGB不适用于图像处理

人眼对于RGB这三种颜色分量的敏感程度是不一样的，在单色中，人眼对红色最不敏感，蓝色最敏感，所以 RGB 颜色空间是一种均匀性较差的颜色空间。如果颜色的相似性直接用欧氏距离来度量，其结果与人眼视觉会有较大的偏差。对于某一种颜色，我们很难推测出较为精确的三个分量数值来表示。

简单来说，如果计算不同颜色之间的对比度，如果使用 RGB 来计算：

(R1-R2)^2 + (G1-G2)^2 + (B1-B2)^2

即使两组颜色数值相同，人的感触还是不一样的，比如这里我选三个颜色：
- RGB_1：110,0,110
- RGB_2：60,0,100
- RGB_3：160,0,110

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0gnnmukt6j20cj0hbt9o.jpg)

可以看到尽管 RGB_1 和 RGB_3 距离 RGB_2 计算的欧式偏差是一样的，但我们还是会明显觉得 RGB_1 相比 RGB_3 更接近 RGB_2 ，因为 RGB_3 看上去比 RGB_1 和 RGB_2 更亮一些。

所以，RGB 颜色空间适合于显示系统，却并不适合于图像处理，图像处理强调的更多是 感触。

### （二）HSV颜色空间

根据颜色的直观特性创建的一种颜色空间，有 A.R. Smith 在 1978年创建的一种颜色空间，也称 六角椎体模型。 

- 色调 Hue
- 饱和度 Saturation
- 明度（亮度）Value

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0gnrdg4m9j219c0rjgoa.jpg)

HSV 对用户来说是一种 直观的颜色模型，我们可以从一种纯色彩开始，即指定色彩角H，并让V=S=1，然后我们可以通过向其中加入黑色和白色，来得到我们需要的颜色。

- 增加黑色可以减小V而S不变
- 同样增加白色可以减少S而V不变

例如：要得到深蓝色：V=0.4，S=1，H=240度。

此外需要额外注意的是，HSV和HSB代指的是同一种颜色空间算法。

### （三）HSL 颜色空间。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0gnxf20hpj20pu0igtab.jpg)

HSV 和 HSL 在字面意思上是一样的：

- H 指的是色相（Hue），就是颜色名称，例如“红色”、“蓝色”；
- S 指的是饱和度（Saturation），即颜色的纯度；
- L（Lightness） 和 V（Value）是明度，颜色的明亮程度

在原理和表现上，HSL 和 HSB 中的 H（色相） 完全一致，但二者的 S（饱和度）不一样， L 和 B （明度 ）也不一样：

- HSV 中的 S 控制纯色中混入白色的量，值越大，白色越少，颜色越纯；
- HSV 中的 V 控制纯色中混入黑色的量，值越大，黑色越少，明度越高
- HSL 中的 S 和黑白没有关系，饱和度不控制颜色中混入黑白的多寡；
- HSL 中的 L 控制纯色中的混入的黑白两种颜色。

### （四）PS上的示例

下面是 Photoshop 和 Affinity Designer 的拾色器。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0go3ts8u6j21470g140u.jpg)

两者分别使用了 HSB 和 HSL 颜色模型。两个拾色器都是 X 轴表示饱和度，越往右，饱和度越高；Y 轴表示明度，越往上明度越高。

先看 Photoshop 的 HSB 颜色模型拾色器，如下图所示，HSB 的 B（明度）控制纯色中混入黑色的量，越往上，值越大，黑色越少，颜色明度越高。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0go4f5ckvj20lv0gijsx.jpg)

如下图所示，HSB 的 S（饱和度）控制纯色中混入白色的量，越往右，值越大，白色越少，颜色纯度越高。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0go4sbvlrj20lw0gp0ub.jpg)

接下来看 Affinity Designer 的 HSL 颜色模型拾色器。如下图所示，Y 轴明度轴，从下至上，混入的黑色逐渐减少，直到 50% 位置处完全没有黑色，也没有白色，纯度达到最高。继续往上走，纯色混入的白色逐渐增加，到达最高点变为纯白色，明度最高。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0go582q20j20j10gewfm.jpg)

### （五）RGB、HSV、HSL转换方程式

```
typedef struct {
    NSUInteger r;
    NSUInteger g;
    NSUInteger b;
    CGFloat a;
} RGB;

typedef struct {
    NSUInteger h;
    CGFloat s;
    CGFloat l;
    CGFloat a;
} HSL;

typedef struct {
    NSUInteger h;
    CGFloat s;
    CGFloat v;
    CGFloat a;
} HSV;

/**
 * Converts an RGB color value to HSL. Conversion formula
 * adapted from http://en.wikipedia.org/wiki/HSL_color_space.
 * Assumes r, g, and b are contained in the set [0, 255] and
 * returns h, s, and l in the set [0, 1].
 *
 * @param   Number  r       The red color value
 * @param   Number  g       The green color value
 * @param   Number  b       The blue color value
 * @return  Array           The HSL representation
 */
HSL RGBToHSL(RGB rgb) {
    CGFloat r = rgb.r / 255.0, g = rgb.g / 255.0, b = rgb.b / 255.0;
    CGFloat max = MAX(MAX(r, g), b), min = MIN(MIN(r, g), b);
    CGFloat h = 0, s = 0, l = (max + min) / 2;

    if (max == min) {
        h = s = 0; // achromatic
    } else {
        CGFloat d = max - min;
        s = l > 0.5 ? d / (2 - max - min) : d / (max + min);

        if (max == r) {
            h = (g - b) / d + (g < b ? 6 : 0);
        } else if (max == g) {
            h = (b - r) / d + 2;
        } else {
            h = (r - g) / d + 4;
        }

        h /= 6;
    }
    return (HSL){ .h = static_cast<NSUInteger>(round(h * 360.0)), .s = s, .l = l, .a = rgb.a };
}

/**
 * Converts an HSL color value to RGB. Conversion formula
 * adapted from http://en.wikipedia.org/wiki/HSL_color_space.
 * Assumes h, s, and l are contained in the set [0, 1] and
 * returns r, g, and b in the set [0, 255].
 *
 * @param   Number  h       The hue
 * @param   Number  s       The saturation
 * @param   Number  l       The lightness
 * @return  Array           The RGB representation
 */
RGB HSLToRGB(HSL hsl) {
    CGFloat h = hsl.h / 360.0, s = hsl.s, l = hsl.l;
    CGFloat r = 0, g = 0, b = 0;

    if (s == 0) {
        r = g = b = l; // achromatic
    } else {
        CGFloat (^hue2rgb)(CGFloat, CGFloat, CGFloat) = ^CGFloat(CGFloat p, CGFloat q, CGFloat t) {
            if (t < 0.0)
                t += 1;
            if (t > 1.0)
                t -= 1;
            if (t < 1 / 6.0)
                return p + (q - p) * 6 * t;
            if (t < 1 / 2.0)
                return q;
            if (t < 2 / 3.0)
                return p + (q - p) * (2 / 3.0 - t) * 6;

            return p;
        };

        CGFloat q = l < 0.5 ? l * (1 + s) : l + s - l * s;
        CGFloat p = 2 * l - q;
        r = hue2rgb(p, q, h + 1 / 3.0);
        g = hue2rgb(p, q, h);
        b = hue2rgb(p, q, h - 1 / 3.0);
    }

    NSUInteger red = round(r * 255);
    NSUInteger green = round(g * 255);
    NSUInteger blue = round(b * 255);

    return (RGB){ .r = red, .g = green, .b = blue, .a = hsl.a };
}

/**
 * Converts an RGB color value to HSV. Conversion formula
 * adapted from http://en.wikipedia.org/wiki/HSV_color_space.
 * Assumes r, g, and b are contained in the set [0, 255] and
 * returns h, s, and v in the set [0, 1].
 *
 * @param   Number  r       The red color value
 * @param   Number  g       The green color value
 * @param   Number  b       The blue color value
 * @return  Array           The HSV representation
 */
HSV RGBToHSV(RGB rgb) {
    CGFloat r = rgb.r / 255.0, g = rgb.g / 255.0, b = rgb.b / 255.0;
    CGFloat max = MAX(MAX(r, g), b), min = MIN(MIN(r, g), b);
    CGFloat h = 0, s = 0, v = max;

    CGFloat d = max - min;
    s = max == 0 ? 0 : d / max;

    if (max == min) {
        h = 0; // achromatic
    } else {
        if (max == r) {
            h = (g - b) / d + (g < b ? 6 : 0);
        } else if (max == g) {
            h = (b - r) / d + 2;
        } else {
            h = (r - g) / d + 4;
        }

        h /= 6;
    }

    return (HSV){ .h = static_cast<NSUInteger>(round(h * 360)), .s = s, .v = v, .a = rgb.a };
}

/**
 * Converts an HSV color value to RGB. Conversion formula
 * adapted from http://en.wikipedia.org/wiki/HSV_color_space.
 * Assumes h, s, and v are contained in the set [0, 1] and
 * returns r, g, and b in the set [0, 255].
 *
 * @param   Number  h       The hue
 * @param   Number  s       The saturation
 * @param   Number  v       The value
 * @return  Array           The RGB representation
 */
RGB HSVToRGB(HSV hsv) {
    CGFloat r = 0, g = 0, b = 0, h = hsv.h / 360.0, s = hsv.s, v = hsv.v;

    NSUInteger i = floor(h * 6);
    CGFloat f = h * 6 - i;
    CGFloat p = v * (1 - s);
    CGFloat q = v * (1 - f * s);
    CGFloat t = v * (1 - (1 - f) * s);

    switch (i % 6) {
        case 0: {
            r = v;
            g = t;
            b = p;
            break;
        }
        case 1: {
            r = q;
            g = v;
            b = p;
            break;
        }
        case 2: {
            r = p;
            g = v;
            b = t;
            break;
        }
        case 3: {
            r = p;
            g = q;
            b = v;
            break;
        }
        case 4: {
            r = t;
            g = p;
            b = v;
            break;
        }
        case 5: {
            r = v;
            g = p;
            b = q;
            break;
        }
    }

    NSUInteger red = round(r * 255);
    NSUInteger green = round(g * 255);
    NSUInteger blue = round(b * 255);

    return (RGB){ .r = red, .g = green, .b = blue, .a = hsv.a };
}
```

------
文章首发：[问我社区](http://www.wenwoha.com/blog_detail-1324.html)

**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)