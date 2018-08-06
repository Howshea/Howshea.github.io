---
title: ViewPager 和 SwipeRefreshLayout 的滑动冲突
date: 2016-10-28 15:16:49
tags: Android
categories: 技术
---
如题，当`SwipeRefreshLayout`包裹`ViewPager`时，发现ViewPager经常滑不动，容易把上面的刷新的小圈圈拽出来，只有手指在屏幕上向斜上方滑或者水平滑动，才能保持正常，这是一个滑动冲突问题。
<!--more-->
## 首先上网查一下别人怎么解决的
好像都是这个解决方案：
```java
viewPager.setOnTouchListener(new View.OnTouchListener() {
      @Override
      public boolean onTouch(View v, MotionEvent event) {
          switch (event.getAction()) {
              case MotionEvent.ACTION_MOVE:
                  swipeRefreshLayout.setEnabled(false);
                  break;
              case MotionEvent.ACTION_UP:
              case MotionEvent.ACTION_CANCEL:
                 swipeRefreshLayout.setEnabled(true);
                 break;
         }
         return false;
     }
});
```
## 问题
嗯，代码看起来很美好，但是运行的时候发现，当下拉的手速比较慢时，刷新不能用了，说明SwipeRefreshLayout没有拦截到事件，然后到ViewPager这儿，就被禁用了。  其实看一下SwipeRefreshLayout的源码就能发现问题：
在SwipeRefreshLayout的构造器里有这么一句
```java	
mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop()
```
然后在`onInterceptTouchEvent()`方法中有这么一段：
```java
final float yDiff = y - mInitialDownY;
    if (yDiff > mTouchSlop && !mIsBeingDragged) {
        mInitialMotionY = mInitialDownY + mTouchSlop;
        mIsBeingDragged = true;
        mProgress.setAlpha(STARTING_PROGRESS_ALPHA);
    }
```
好了，问题如此清晰明了，当我们下拉的时候比较慢，第一次手指在y轴滑动的距离还不够`mTouchSlop`那么长，所以没有拦截，那viewpager秒秒钟给它禁掉了，因为我们在上面重写了ViewPager的触摸事件，之后yDiff的距离够长了，但是它已经被禁掉啊，然后就会出现只有一个半透明的小球滑下来，然后收上去了，什么也没发生……
## 解决方案
其实，这个滑动冲突的问题的关键在于，当手指向斜下方滑动时，手指在屏幕上移动的x轴的距离大于y轴的距离，这个时候我们是想让viewpager响应的，那重写SwipeRefreshLayout的`onInterceptTouchEvent()`方法就好了啊，当滑动X轴的距离大于Y轴的距离就让SwipeRefreshLayout不要拦截事件，完整的代码如下：
```java
public class ReformSwipRefreshLayout extends SwipeRefreshLayout {
    private float startY;
    private float startX;
    // 记录viewPager是否拖拽的标记
    private boolean mIsDraggingFlag;
    private final int mTouchSlop;

    public ReformSwipRefreshLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                startY = ev.getY();
                startX = ev.getX();
                // 初始化标记
                mIsDraggingFlag = false;
                break;
            case MotionEvent.ACTION_MOVE:
                // 如果子view正在拖拽中，则不拦截
                if(mIsDraggingFlag) {
                    return false;
                }
                float endY = ev.getY();
                float endX = ev.getX();
                float distanceX = Math.abs(endX - startX);
                float distanceY = endY - startY;
                // 如果X轴位移大于Y轴位移或者Y轴位移为负数时，事件交给子View处理
                if(distanceX > mTouchSlop && distanceX>distanceY) {
                    mIsDraggingFlag = true;
                    return false;
                }
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                // 初始化标记
                mIsDraggingFlag = false;
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }

}
```