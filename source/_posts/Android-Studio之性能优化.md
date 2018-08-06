---
title: Android Studio 之性能优化
date: 2016-04-06 16:59
tags: 
  - Android
  - Android Studio
categories: 技术
---
为自己的笔记本更换SSD已有一个月的时间，当时换ssd的主要原因是因为Android Studio太卡，然而换上ssd之后，虽然好了很多，但是仍然觉得studio运行得不理想，郁闷呐，好歹我上的也是一块高端ssd。今天上课的时候上知乎查了一下怎么优化，竟然一下子查到了，还是知乎给力啊(奇怪的是我之前也查过，不知道为什么没查到),优化过程如下：
<!--more-->  
# <small>修改studio.exe.vmoptions和studio64.exe.vmoptions文件</small>  
在AndroidStudio目录下bin文件夹下找到两个文件`studio.exe.vmoptions`和`studio64.exe.vmoptions`
![](http://ww3.sinaimg.cn/large/8127619ajw1f2n6gry45pj20hd032t8q.jpg)
默认的参数分配的内存很少，我的笔记本8G内存，不利用起来是严重的浪费  
>-Xms2048m  
>-Xmx2048m  
>-XX:MaxPermSize=2048m  
>-XX:ReservedCodeCacheSize=1024m  
  
如上，这几个参数都改大一点就行，我改成这样启动速度已经快得飞起，幸福感爆棚了  
然后再加一行：
>-XX:+UseCompressedOops  
  
这句用于压缩指针空间，可以减少一定的内存占用(64位才支持)  
# <small>修改AndroidStudio设置</small>  
勾选offline work  
![](http://ww4.sinaimg.cn/large/8127619agw1f2n5pcjy2sj20un0jbq3x.jpg)  
  
最后，尽量使用新版Android Studio，新版的明显性能更好！