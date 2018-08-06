---
title: 解决 Activity 跳转时动画卡住的问题
date: 2016-05-15 13:01:49
tags: Android
categories: 技术
---
昨天开始着手做一个小项目，一开始就遇到了一个头疼的问题。  
我在ToolBar的右边添加了一个ImageButton，用来代替三个点的菜单按钮，样子如下图
<!--more-->
![](http://ww4.sinaimg.cn/mw690/8127619agw1f3w0ekmb4sj20xm0eljy2.jpg)
我为这个小按钮写了一个翻转动画，即点一下旋转180度并弹出popupMenu，其它操作旋转回来。
主要代码如下：  
```java
case R.id.settings:  
   //先执行一个翻转动画  
   animation0.setDuration(200);  
   animation0.setFillAfter(true);  
   settings.startAnimation(animation0);  
   PopupMenu popupMenu = new PopupMenu(MainActivity.this, v);  
   popupMenu.inflate(R.menu.main_menu);  
   popupMenu.setOnMenuItemClickListener(MainActivity.this);  
   popupMenu.setOnDismissListener(MainActivity.this);  
   popupMenu.show();  
   break;  
}  
......  
@Override  
public boolean onMenuItemClick(MenuItem item) {  
   switch (item.getItemId()) {  
       case R.id.setting:  
            startActivity(new Intent(this, PreferenceActivity.class));
            return true;  
       case R.id.about:  
            ......  
            return false;  
       default:  
            return false;  
   }  
}  
......  
@Override
public void onDismiss(PopupMenu menu) {  
   animation1.setDuration(200);  
   animation1.setFillAfter(true);  
   settings.startAnimation(animation1);  
}  
```

    

现在出现的问题是:  
**当我点击menu中的设置跳转到设置界面时，那个按钮的动画会被卡住，等我从设置页面回来，动画还会走完剩下的一半**，所以，特别难看。  
  
出现这个问题，无非是动画走到一半就被当前Activiy的`onPause()`方法打断了，于是我在`startActivity()`加了一句：  
> settings.clearAnimation();  
  
结果，竟然无效，这就很奇怪了。  
  
于是做了一个测试：  
在`onMenuItemClick()`,`onDismiss()`和`onPause()`中分别加一句：  
> System.out.println("onMenuItemClick");  
> System.out.println("onDismiss");  
> System.out.println("onPause");  
  
运行结果：  
![](http://ww4.sinaimg.cn/large/8127619agw1f3w0el8k1lj20ky01m3ys.jpg)  
  
这下问题就很清晰了，popupMenu的回调方法是先响应**onMenuItemClick()**,后响应**onDismiss()**,所以不管之前做什么操作，这个动画都会执行，那解决方案也十分明确了，由于**onPause()**是最后才执行，所以重写Activity的`onPause()`方法即可：  
```java  
@Override  
protected void onPause() {   
   settings.clearAnimation();  
   System.out.println("onPause");  
   super.onPause();  
}  
```

