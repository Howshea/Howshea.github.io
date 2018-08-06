---
title: Kotlin 中 Lambda 表达式的典型实践之一
date: 2017-06-05 09:40:08
tags: 
  - Android
  - Kotlin  
categories: 技术
---
学习一下使用 Kotlin 的 Lambda 表达式简化 Android 中的 setOnClickListener
<!--more-->
# Lambda 表达式
在 Android 迟迟不能使用 Java8 的情况下，Kotlin 支持 Lambda 表达式则显得非常有用。  
Lambda 表达式实际就是匿名函数，但是它仍然属于表达式

# Java 的方式实现 view 点击事件
```java
view.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v) {
    	Toast.makeText(
           v.getContext(), 
           "Click", 
           Toast.LENGTH_SHORT).
        show();
    }
})
```
老司机都能看懂这一段哈，很简单，接下来把上面的 Java 代码换成 Kotlin
# Kotlin 实现 view 的点击事件
## 普通实现方法
(这里使用了 Kotlin 第三方库 Anko 的 toast 函数)
```kotlin
view.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View) {
       toast("Click")
    }
})
```
看起来简单了一些，但是实际没省多少事，毕竟用 Java 封装一下 Toast 也能简化一点
## Lambda 实现方法
在Android Studio 3.0 中，当输入 `setOnClickListener()` 之后可以看到一个参数的提示，IDE提示允许接收 `I:((v:View!)->Unit)!` 这样的一个参数，这就是一个 lambda 表达式，Unit 代表没有返回值，所以上面的代码我们可以写成：
```kotlin
view.setOnClickListener({view -> toast("Click")})
```
另外，Kotlin 允许把函数的最后一个 Lambda 表达式参数移到小括号外，所以上面的语句可以写成：
```kotlin
view.setOnClickListener(){view -> toast("Click")}
```
不止如此，如果函数只有一个 Lambda 表达式,小括号也可以不写，唯一一个参数也可以省略，这样：
```kotlin
view.setOnClickListener{toast("Click")}
```
比Java的实现代码短了很多，而且具有一种简洁的美感。