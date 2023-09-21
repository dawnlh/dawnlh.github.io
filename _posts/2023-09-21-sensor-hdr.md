---
title: 图像传感器中的 HDR 技术
date: 2023-09-20 11:33:00 +0800
categories: [Optics & Imaging]
tags: [hdr]
math: true
img_path: /assets/img/post_img/2023-09-21-sensor-hdr/
---

动态范围是成像中非常重要的一个性能指标，它描述了在一幅图像里能够同时体现高光和阴影部分内容的能力。如果动态范围比较窄，很容易出现亮处过曝或者暗处全黑的不理想效果。

![](20230920-201132.png)

动态范围的数学定义为：

$$
DR = 20log_{10}\frac{i_{max}}{i_{min}}
$$

其中，$i_{max}$ 是 sensor 的最大不饱和电流，也可以说是 sensor 刚刚饱和时候的电流，$i_{min}$ 是 sensor 的底电流 (blacklevel）。动态范围的单位为 dB，人眼动态范围可以到 100dB 左右，高端相机的动态范围一般大于 80dB。

为了提高拍摄时的亮度采样范围和输出图像的亮度层次感，高动态范围（HDR，也有叫宽动态范围，WDR）成为成像中的一个核心需求。通过 Sensor 设计来提高成像的动态范围的主要思路包括：多帧融合、长短曝光融合、大小像素融合等。

## 单帧 HDR

单帧 HDR 是指通过一次曝光采集就可以实现 HDR 的技术，即在采集到的一帧图像中同时包含了曝光亮不同的亮部信息和暗部信息，通过融合得到 HDR 图像。其优势是避免了多帧采集再融合的 HDR 方案中，由于不同帧采集时刻不同，而可能出现的内容不对齐、运动模糊的现象。其缺点在于，大部分的单帧 HDR 方案都是以牺牲空间分辨率为代价的。

### Quad Bayer HDR

这个应该算是目前比较成熟且相对主流的 HDR 方案之一了，在很多手机 sensor 上都有采用。典型方案如 Sony IMX689 和最新的 IMX989 等 Sensor 所采用的 [QBC](https://www.sony-semicon.com/cn/technology/mobile/quad-bayer-coding.html)（quad bayer coding）技术。

![](20230920-113004.png)


![](20230920-113645.png)


这种 sensor 的设计是：每个像素是有四个子像素组成，他们共用一个 color filter。

非 HDR 模式/弱光模式下，将同一个 color filter 下的 4 个相邻像素的信号相加后再输出，相当于增大了像素的靶面，能够感光度，进而提高信噪比，改善夜景弱光成像的效果。

在 HDR 模式时，会把四个像素分成两组，对角线方向的分到同一组，135 度的那组曝光要短于 45 度的那组，同一组两个像素值相加后输出，这样一次曝光可以采集到两张曝光时间不同的图像，对二者进行融合就可以生成 HDR 图像。

OmniVision 的 [OV48B](https://www.ovt.com/products/ov48b/) sensor 也采用了类似的 HDR 方案，叫作 4-Cell（名字过于简单直接）。它和 Sony 方案的区别在于，它除了支持按对角线 1+4,2+3 方式将像素分为两组外，还可以按照 1+2+3,4 的方式把右下角和其他三个像素分为两组。

具体的曝光分组方案还有更多不同的设计，比如 [Sony IMX586](https://www.sony.com/en/SonyInfo/News/Press/201807/18-060E/) 支持 3HDR，即 4 个像素分了三种不同的曝光时长。三星也有类似的 3HDR 方案。

### Interlaced HDR

![](20230920-171149.png)


隔行曝光 HDR，典型方案如 Sony IMX135 和 IMX258 采用的 BME (binned multiplexed exposure) HDR 技术和 OV 采用 Alternate row HDR 技术。

![](20230920-115619.png)


这种方案的思路是，相邻两行为一组（因为 bayer 阵列的结构一般是两行为一组），相邻两组一组做长曝光，一组做短曝光。通过这种方式采集到的图像，在一帧中就同时包含了长曝光和短曝光的行。算法最后会进行融合处理来得到一张 HDR 的图像。整体上看，相当于是通过 1×2 的 binning 牺牲一半分辨率，换取两帧不同 HDR 的图形。

### Zig-zag HDR

![](20230920-121335.png)


Zig-zag HDR 基本思想源自 Interlaced HDR，但做了改进。它不再以行为单位做曝光时间区分，而是按照 Z 字形方式组织长曝光和短曝光的数据。

### SME HDR

> Spatial Multiplexed Exposure

![](20230920-114918.png)

dawnlh/dawnlh.github.io/_posts/20230920/20230920-113004.png
Sony IMX214 Sensor 采用了这一技术。这种 sensor 在空间上以棋盘格的 pattern 排列着长曝光和短曝光的像素，具体排布方式没有公布。采集到的图像会通过算法处理融合成为一帧 HDR 图像。按照 Sony 的说法，这种 SME 技术只损失 20% 的空间分辨率。对于这种方案来说，ISP 融合算法的好坏至关重要。

### 大小像素

![](20230920-165236.png)


除了上述的长短曝光方案外，还有一类方案是通过在 Sensor 上集成不同大小的像素来实现 HDR。

![](20230920-184105.png)


比如 [Samsung ISOCELL Auto 4AC](https://semiconductor.samsung.com/cn/image-sensor/automotive-image-sensor/isocell-auto-4ac/) 车载 CIS（CMOS image sensor）采用的 CornerPixel™ 技术中，每个像素都使用两个光电二极管来处理每个画面，一个 3.0 微米的大像素用于处理暗色区域，一个 1.0 微米的小像素用于处理亮色部分，两个不同大小的像素 sensitivity 不同，满井容量（full-well capacity，FWC）也不同，搭配起来，能够使传感器实现 120dB 的 HDR 和极低的运动模糊。

### Dual Gain HDR

在 CMOS 感光元件中，我们可以通过设置其增益值（模拟增益和数字增益）来放大信号。一般相机都会有一个 ISO（感光度）调节功能，能够实现信号增益的控制。

传统的相机在放大信号同时，噪声也一起放大了。可不可以通过在不同曝光条件下使用不同增益改善信号质量，辅助实现 HDR 呢？答案是有的，目前 sensor 设计里有两种主要的双增益方法: 一种在像素里做，叫做 DCG（Dual Conversion Gain）；一种在读出电路里做，叫做 DGA（Dual Gain Amplifier）

#### DCG

> Dual Conversion Gain

DCG 是像素级的双增益方法，它将增益分出了两档：High Conversion Gain（HCG）和 Low Conversion Gain（LCG）。LCG 模式 应对明亮的场景，灵敏度（增益）较低，HCG 模式 应对低光场景，灵敏度（增益）增加。

![](20230920-185331.png)


上图是 DCG 技术的像素电路，和传统的像素电路对比，最大的不同是红框处。这里增加了一个 DCG 开关。当处于高光照环境时，DCG 开关打开，连接 FD 和物理电容，FD 节点电容变大，这时对应 LCG 模式。当处于阴暗光照环境时，DCG 关闭，断开 FD 和物理电容，这时对应 HCG 模式。这种情况下，灵敏度变高，读出噪声变小。

#### DGA

> Dual Gain Amplifier

![](20230920-185823.png)


DGA 实际电路中有两路放大器，而且这个放大器不是 DCG 里面那种像素级的。上图给出了一个例子，是 Neo 和 Zyla 相机 [sCOMS sensor](https://andor.oxinst.com/learning/view/article/dual-amplifier-dynamic-range) 中使用的双增益放大器电路，它里面是用了两个 column-level 的放大器，对应不同增益。

DGA 的目的和 DCG 类似，只是对于 DCG 方式来讲，如果没有特殊处理，每一帧只能选择 HCG 或者 LCG 中的一种；而对 DGA 来说，可以做成一帧内同时读出 HCG 和 LCG 数据。

## 多帧 HDR

多帧 HDR 方案是通过连续采集不同曝光时间的两帧或者多帧图像进行融合，从而获得 HDR 图像的技术。相比单帧 HDR 方案，它一般不会损失空间分辨率，但是会损失时间分辨率。由于不同曝光时长的帧在时间上没有对齐或者整体帧率下降，可能会导致运动模糊现象。

### Frame-based HDR

![](20230920-174537.png)


这种是最直接的多帧 HDR 方案，即先采集一帧长曝光的图像并读出，然后再采集一帧短曝光的图像并读出，循环进行此过程。在 ISP 中会对长短曝光图像进行融合，得到 HDR 图像。这种方式的缺点也很明显，就是必须等长曝光图像采集完成后才能采集短曝光图像，这会导致两帧之间有明显时差，而且在融合后输出视频的整体帧率会掉一半，所以对运动目标成像时可能产生运动模糊。

### Line Interleaving HDR

> 行交织 HDR

![](20230920-180158.png)


一般常见的 CMOS sensor 的快门都是滚动快门（也叫卷帘快门，Rolling Shutter），其曝光和读出都是从上至下一行行读取的，而不是曝光结束后一次性读取的（这种叫作全局快门，global shutter）。

针对这种滚动快门的 Sensor，可以采用行交织的方式采集长短曝光的图像，即在每一行进行完一次长曝光后，立即再进行一次短曝光，而不是像 frame-based HDR 那样等所有行都长曝光结束读出一帧之后再采集短曝光，这样可以大大减小长短曝光的时差。从 readout 角度来看，就是长曝光帧与短曝光信号交错着依次输出，从而达到“准同时”输出多帧不同曝光时间图像的效果，能有效缓解 frame-based HDR 中存在的运动模糊问题。

不过这种方案也会带来一些额外的问题。由于它需要对于每一行进行多重曝光，每一行的曝光时间的长度实际上就是各次曝光时间的总和，这样会导致各行之间曝光开始的时间差远远超过了单次曝光模式下的行时间差。这一点在 CMOS Sensor 的 Rolling Shutter 下，会使得果冻效应更加明显。这一点是由其完整的工作原理所限定，无法进行优化。不过果冻效应一般也就是在光线很暗曝光时间较长，以及针对高速运动的物体进行拍摄的时候才会出现，所以对于 DOL HDR 模式的使用而言，就需要尽可能避免这种应用场景。

基于 Line Interleaving HDR 方案，SONY 实现的技术叫作 DOL-HDR（Digital Overlap HDR）在 IMX335 等 Sensor 上支持该模式；OV 实现方案为 staggered HDR。DOL-HDR 最多支持 4:1 曝光输出 (long, medium, short, very short)， OV 最多支持 3:1 输出 (long, medium, short)

## 总结

总结起来，从时间上来说，增强 HDR 的手段可以分为单帧和多帧 HDR，两者各有优势。前者主要优点在于基本没有运动模糊，但是一般会损失空间分辨率；后者一般不会损失空间分辨率，但是可能损失时间分辨率，导致运动模糊。在实际应用中，不同类型的 HDR 技术有的可以结合使用，来获得更大的动态范围。

## Reference

- [SONY HDR sensor 简介 - 知乎](https://zhuanlan.zhihu.com/p/37374601)
- [SONY HDR sensor 和 HDR 技术名词简介 - 知乎](https://zhuanlan.zhihu.com/p/348338750)
- [Camera和Image sensor技术基础笔记(5) -- HDR相关技术\_hdr sensor\_亦枫Leonlew的博客-CSDN博客](https://blog.csdn.net/vivo01/article/details/124173495)
- [HDR sensor 原理介绍 - 吴建明wujianming - 博客园](https://www.cnblogs.com/wujianming-110117/p/12684011.html)
- [diezemann.com/wp-content/uploads/2020/06/20200312-IS-Europe-Dana-Diezemann-HDR-Methods-Final.pdf](https://diezemann.com/wp-content/uploads/2020/06/20200312-IS-Europe-Dana-Diezemann-HDR-Methods-Final.pdf)
