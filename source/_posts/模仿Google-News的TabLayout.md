---
title: 模仿Google News 的 TabLayout
date: 2019-04-22 11:16:32
tags:
  - Android
categories: 技术
---

## 前景提要
很久之前看到 Google News 的 `TabLayout` 的样式挺有意思的，如下图：

<!--more-->

![](https://user-gold-cdn.xitu.io/2019/1/8/1682b5db9a2db76c?w=1080&h=2160&f=png&s=1088210)
具体效果大家可以自己下载 Google News 看一下，截图上大概看出来一共有两个要素：

1. 指示器和文字宽度相同
2. 指示器的形状是半个圆角矩形

于是，我模仿的结果：（截图来自我的一个小项目里[GeekNews](https://github.com/howshea/GeekNews)）
![](https://user-gold-cdn.xitu.io/2019/1/8/1682b79ea8ea9c3e?w=1080&h=2160&f=png&s=424859)

开始模仿之前，先问个问题，这个控件是 TabLayout 吗？答案：是的，我用 monitor 看过了。  
所以可以得到结论：直接魔改源码是最简单最快的方法。
## 实现思路
### 魔改系统组件的第一步都是先把源码拷出来
![](https://user-gold-cdn.xitu.io/2019/1/8/1682b7d7ca8d4b40?w=382&h=240&f=png&s=26774)
就这四个文件，拷出来改改里面一些类的引用路径，试一下能用就行了

### 实现半个圆角矩形
简单看一下 TabLayout 这个类的结构可以看出， TabLayout 内的指示器是由一个 私有内部类 `SlidingTabStrip` 来控制的，再看一下其中的 `draw` 方法实现，
```java
public void draw(Canvas canvas) {
    super.draw(canvas);
    // Thick colored underline below the current selection
    if (mIndicatorLeft >= 0 && mIndicatorRight > mIndicatorLeft) {
        canvas.drawRect(mIndicatorLeft, getHeight() - mSelectedIndicatorHeight,
mIndicatorRight, getHeight(), mSelectedIndicatorPaint);
    }
}
```
这里画了一个矩形，从`mIndicatorLeft`、`mIndicatorRight`、`mSelectedIndicatorPaint`这几个变量的名字上就非常非常明显可以看出，这个矩形就是tab下面的那个矩形指示器条 (此处应有️配图)

这个地方我们可以先实现半个圆角矩形的效果，圆角很简单，把`drawRect` 换成 `drawRoundRect` 即可，半个矩形只要画一个超过控件最底部的rectF,让控件自己裁掉这个rectF的一半高度，圆角则取这个一半高度，就是`mSelectedIndicatorHeight`的值
```java
public void draw(Canvas canvas) {
    super.draw(canvas);
    // Thick colored underline below the current selection
    if (mIndicatorLeft >= 0 && mIndicatorRight > mIndicatorLeft) {
        RectF rectF = new RectF(mIndicatorLeft, getHeight() - mSelectedIndicatorHeight, mIndicatorRight, getHeight() + mSelectedIndicatorHeight);
        mSelectedIndicatorPaint.setAntiAlias(true);
        canvas.drawRoundRect(rectF, mSelectedIndicatorHeight, mSelectedIndicatorHeight, mSelectedIndicatorPaint);
    }
}
```
可以看到这里我们仅利用已有的变量就能实现半个圆角矩形的效果。

### 实现指示器宽度与文字等宽
从上一步我们可以看到，指示器的宽度是由 `mIndicatorLeft` 和 `mIndicatorRight` 这两个变量决定的，那直接在draw方法里改？显然不行，想想，tab切换涉及两个tab的宽度计算，`mIndicatorLeft` 和 `mIndicatorRight`的计算不仅跟着位置改变还受到tab本身宽度的影响（其实我偷偷试过了，在这里改确实不行）。  
首先，我们要找到 `mIndicatorLeft` 和 `mIndicatorRight` 被修改的地方，发现一个方法：
```java
void setIndicatorPosition(int left, int right) {
    if (left != mIndicatorLeft || right != mIndicatorRight) {
        // If the indicator's left/right has changed, invalidate
        mIndicatorLeft = left;
        mIndicatorRight = right;
        ViewCompat.postInvalidateOnAnimation(this);
    }
}
```
我们顺着这个方法被调用的地方终于找到 left 和 right 计算的地方：
```java
private void updateIndicatorPosition() {
    final View selectedTitle = getChildAt(mSelectedPosition);
    int left, right;
    if (selectedTitle != null && selectedTitle.getWidth() > 0) {
        left = selectedTitle.getLeft();
        right = selectedTitle.getRight();
        if (mSelectionOffset > 0f && mSelectedPosition < getChildCount() - 1) {
            // Draw the selection partway between the tabs
            View nextTitle = getChildAt(mSelectedPosition + 1);
            left = (int) (mSelectionOffset * nextTitle.getLeft() + (1.0f - mSelectionOffset) * left);
            right = (int) (mSelectionOffset * nextTitle.getRight() + (1.0f - mSelectionOffset) * right);
        }
    } else {
        left = right = -1;
    }
    setIndicatorPosition(left, right);
}
```
终于找到修改的地方了，首先我们要了解一下第一行 getChildAt 返回的 view 是个什么，这里就不贴代码了，直接说结论：阅读源码可知是个名为 TabView 的类，TabView 是个 LinearLayout，TabLayout 的文字是由其内部的mTextView来显示的。计算的思路就是下面的灵魂示意图：

![](https://user-gold-cdn.xitu.io/2019/1/8/1682c8f07936ff24?w=1486&h=930&f=png&s=95116)
翻译成伪代码就是：
```
spacing = (view.width -view.mTextView.width)/2
newLeft = view.left + spacing
newRight = view.right - spacing
```
所以修改之后的 `updateIndicatorPosition()` 方法如下，要注意需要修改两组left和right，一个是当前选中的Tab，一个是下一个Tab
```java
private void updateIndicatorPosition() {
    final TabView selectedTitle = (TabView) getChildAt(mSelectedPosition);
    int left, right;

    if (selectedTitle != null && selectedTitle.getWidth() > 0) {
        int spacing = (selectedTitle.getWidth() - selectedTitle.mTextView.getMeasuredWidth()) / 2;
        left = selectedTitle.getLeft() + spacing;
        right = selectedTitle.getRight() - spacing;

        if (mSelectionOffset > 0f && mSelectedPosition < getChildCount() - 1) {
            // Draw the selection partway between the tabs
            TabView nextTitle = (TabView) getChildAt(mSelectedPosition + 1);
            int nextSpacing = (nextTitle.getWidth() - nextTitle.mTextView.getMeasuredWidth()) / 2;
            int nextLeft = nextTitle.getLeft() + nextSpacing;
            int nextRight = nextTitle.getRight() - nextSpacing;

            left = (int) (mSelectionOffset * nextLeft +
                            (1.0f - mSelectionOffset) * left);
            right = (int) (mSelectionOffset * nextRight +
                            (1.0f - mSelectionOffset) * right);
        }
    } else {
        left = right = -1;
    }

    setIndicatorPosition(left, right);
}
```
这就修改完了吗？不！要知道tab之间切换有两种方式，一个是 viewPager 划过去，还有一个是点击任意一个tab跳过去，所以还有一个地方要改，不需要找了，setIndicatorPosition 紧跟着的下一个方法就是，`void animateIndicatorToPosition(final int position, int duration)` ，我们只要改其中的这一段：
```java
final View targetView = getChildAt(position);
...
final int targetLeft = targetView.getLeft();
final int targetRight = targetView.getRight();
```
只需要改目标 view 的 left 和 right，因为这个方法里调用了一次 `updateIndicatorPosition()`，当前选中的 view 已经被计算过一次了。
修改后：
```java
final TabView targetView = (TabView) getChildAt(position);
...
int targetSpacing = (targetView.getWidth() - targetView.mTextView.getMeasuredWidth()) / 2;
final int targetLeft = targetView.getLeft() + targetSpacing;
final int targetRight = targetView.getRight() - targetSpacing;
```
至此，真的就全部完成了。
修改好的源码地址：[TabLayout.java](https://github.com/howshea/GeekNews/blob/master/basemodule/src/main/java/com/howshea/basemodule/component/viewGroup/tabLayout/TabLayout.java)
![](https://user-gold-cdn.xitu.io/2019/1/8/1682cccd096faf2b?w=240&h=240&f=jpeg&s=9839)