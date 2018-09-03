---
title: 数学在UI上的两个应用
date: 2018-09-03 09:14:36
tags:
   - UI
   - 数学
categories: 学习
---

前几天听了一期设计师的播客节目，说到高斯模糊和投影的衰减，想起自己恰好研究过这两个问题，又回忆起池老师说的闭环原则，觉得有必要记录和分享出来。

偷偷推荐一下这个播客，这期播客链接在这里:  [№ 71[学好数理化，做遍设计都不怕]](https://anyway.fm/science-in-ui-design/#title)

<!--more-->

# 高斯模糊

高斯模糊是一种在移动端常见的设计，俗称毛玻璃效果，当然，在软件上我们难以模拟一个毛玻璃的光线效果，所以，高斯模糊是直接对图像进行处理。

先说一下模糊算法，简单来说，就是对每个像素的周围的像素取平均值之后重新设置这个像素的颜色

![](http://wx4.sinaimg.cn/mw690/8127619agy1fuw4kfc9z7j20az096mwy.jpg)

我们常用的 blur 算法里要传的那个参数，即是周围取值的半径，如上图所示，半径为 1 时，只取周围一个像素的范围。

那取平均值和高斯有什么关系，实际上直接取平均值做出来的效果并不自然，因为图像是连续的，越靠近的像素关联度越高，所以这个平均值应该是一个加权平均值，而权重的计算符合高斯分布，说高斯分布可能多少人不熟，因为我们读高中的时候都叫它「正态分布」。

![](http://wx1.sinaimg.cn/mw690/8127619agy1fuw4xsmfdnj20go0bowez.jpg)

以上高斯模糊的原理，就基本解释明朗了，下面说一下高斯模糊在实际开发里的 best practice。

我们已经知道了高斯模糊的计算方式，拿一个 1080 \* 1920 px 的图片为例，像素点超过 200百万个，而且为了模糊效果平滑，我们通常会把半径设置的较大，一般是10，这个计算量是恐怖的，即使在高端机上也会有明显的卡顿，内存小的机器甚至会由于频繁GC造成处理失败，所以我通常的做法是先对图片做缩小处理，比如把 1080P 的图像缩小三倍，变成 360 \* 640 px，这样计算量就小了至少10倍。

# 投影衰减

投影效果在 UI 上应用广泛，在 google 的材料化设计中更是最重要的元素之一，但是无论设计还是开发遇到过一个头疼的问题，就是投影效果的制作，因为自然的投影衰减并不是线性的。

![](http://wx2.sinaimg.cn/mw690/8127619agy1fuw643votsj20zd0lfjst.jpg)

我们想要渐变的实际是应该符合上图中红色的那个函数，但是软件只能做出符合蓝色的线性函数的渐变。

上面播客里主播有提到他们是用多层投影来模拟非线性的渐变（类似于求极限的思路）。在 Android 开发中，shape 也只能制作线性渐变，效果很不自然，而控件自带投影效果，5.0以下的系统实现不了，5.0以上的也不是所有组件都可以使用，而且不能调整颜色。

所幸的是，14年的时候一位 google 工程师就写出了相应的投影渐变算法: [plus.google.com](https://plus.google.com/+RomanNurik/posts/2QvHVFWrHZf), 文中的算法链接已经失效了，因为他把算法用 Kotlin 重新实现了一遍。我把算法直接记录在这篇 blog 里，方便其他人阅读。

```kotlin
private val cubicGradientScrimCache = LruCache<Int, Drawable>(10)

fun Float.constrain(min: Float, max: Float): Float = Math.max(min, Math.min(max, this))

/**
 * Creates an approximated cubic gradient using a multi-stop linear gradient. See
 * [this post](https://plus.google.com/+RomanNurik/posts/2QvHVFWrHZf) for more
 * details.
 */
@SuppressLint("RtlHardcoded")
@JvmOverloads
fun makeCubicGradientScrimDrawable(
    gravity: Int,
    alpha: Int = 0xFF,
    red: Int = 0x0,
    green: Int = 0x0,
    blue: Int = 0x0,
    requestedStops: Int = 8
): Drawable {
    var numStops = requestedStops

    // Generate a cache key by hashing together the inputs, based on the method described in the Effective Java book
    var cacheKeyHash = Color.argb(alpha, red, green, blue)
    cacheKeyHash = 31 * cacheKeyHash + numStops
    cacheKeyHash = 31 * cacheKeyHash + gravity

    val cachedGradient = cubicGradientScrimCache.get(cacheKeyHash)
    if (cachedGradient != null) {
        return cachedGradient
    }

    numStops = Math.max(numStops, 2)

    val paintDrawable = PaintDrawable().apply {
        shape = RectShape()
    }

    val stopColors = IntArray(numStops)

    for (i in 0 until numStops) {
        val x = i * 1f / (numStops - 1)
        val opacity = Math.pow(x.toDouble(), 3.0).toFloat().constrain(0f, 1f)
        stopColors[i] = Color.argb((alpha * opacity).toInt(), red, green, blue)
    }

    val x0: Float
    val x1: Float
    val y0: Float
    val y1: Float
    when (gravity and Gravity.HORIZONTAL_GRAVITY_MASK) {
        Gravity.LEFT -> {
            x0 = 1f
            x1 = 0f
        }
        Gravity.RIGHT -> {
            x0 = 0f
            x1 = 1f
        }
        else -> {
            x0 = 0f
            x1 = 0f
        }
    }
    when (gravity and Gravity.VERTICAL_GRAVITY_MASK) {
        Gravity.TOP -> {
            y0 = 1f
            y1 = 0f
        }
        Gravity.BOTTOM -> {
            y0 = 0f
            y1 = 1f
        }
        else -> {
            y0 = 0f
            y1 = 0f
        }
    }

    paintDrawable.shaderFactory = object : ShapeDrawable.ShaderFactory() {
        override fun resize(width: Int, height: Int): Shader {
            return LinearGradient(
                width * x0,
                height * y0,
                width * x1,
                height * y1,
                stopColors, null,
                Shader.TileMode.CLAMP)
        }
    }

    cubicGradientScrimCache.put(cacheKeyHash, paintDrawable)
    return paintDrawable
}
```

# 总结

今年的第二篇 blog，依旧没什么技术含量。