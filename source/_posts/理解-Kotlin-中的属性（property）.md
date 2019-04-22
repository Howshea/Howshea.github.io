---
title: 理解 Kotlin 中的属性（property）
date: 2019-04-22 11:20:23
tags:
  - Kotlin
  - Android
categories: 技术
---

> 这篇文章是一时兴起想写的，因为我发现我对Kotlin的属性理解一直有误

<!--more-->

## Java 中的属性是什么（property）
首先我们要搞清楚在 Java 中属性是什么，在 Java 中类的属性不是指一个字段，而是一个字段和它的get、set方法加在一起才算一个属性，比如：

```java
class Person {
    int age;
    public int getAge(){
        return age;
    }
    public void setAge(int age){
        this.age = age
    }
}
```

而如果不给这个 name 写get、set方法，它就只是一个 **field**,可以称它为 **字段** 或者 **域**。

## Kotlin的类只有属性（property）没有独立的字段（field）
如果上面的代码用kotlin翻译一下：

```kotlin
class Person {
    var age = 0
}
```

可以对比出，Kotlin里不需要额外的get、set方法，当然你也写不了，因为 Kotlin 已经默认实现了get、set，所以在Kotlin里，我们写不出 **field**。

## Kotlin 中的 backing field 是什么
前面说 Kotlin 中不能写 field，但是我们很多时候必须要用到 field，比如你复写一个属性的set方法

```kotlin
var age = 0
    set(value){
        age = value + 1
    }
```

如果这样写其实是会发生递归，无法赋值成功
![](https://user-gold-cdn.xitu.io/2019/4/12/16a0f72d171968dc?w=432&h=198&f=png&s=20920)
这里AS也提醒你了，这里发生了递归  
所以我们一般都这么写：
```kotlin
var age = 0
    set(value){
        field = value + 1
    }
```
这里出现了一个 **field** 的东西可以访问和赋值，这个东西就是 Kotlin 的**幕后字段（Backing Field）**  
我们可以简单地理解为，Kotlin 没有明面上的 field，但是它存在于幕后

## 没有 backing field的成员是什么？

```kotlin
class Person{
    private var _age =0
    var age:Int
        get() = _age
        set(value) {
            _age = value
        }
}
```

当我们声明一个 var 为私有时，比如上面的_age，我们叫它 **幕后属性** ,虽然它看起来不需要get、set方法，但是其实仍然是有的，只不过它的get、set方法都被声明为 private 了

**当然我们这里不是想讨论幕后属性**，而是要讨论一下这个 age 是个什么玩意，是一个成员属性吗，其实通过字节码就可以知道，这里的Person类实际并不存在age这个成员，它只是帮你生成了对_age的两个public final的get和set方法

```
// access flags 0x2
private I _age

// access flags 0x11
public final getAge()I
 L0
  LINENUMBER 28 L0
  ALOAD 0
```
和你自己直接写一个

```kotlin
fun getAge() = _age
```
是一模一样的  

**再举一个例子**：

```kotlin
var View.topPadding: Int
    inline get() = paddingTop
    set(value) = setPadding(paddingLeft, value, paddingRight, paddingBottom)
```

这是从anko的拷出来一段代码，通过这个扩展成员，我们可以直接对某个 View 的 PaddingTop 进行修改和读取，虽然说是成员，但是我们把一段字节码拿出来看一下：

```
// access flags 0x19
public final static getTopPadding(Landroid/view/View;)I
  @Lorg/jetbrains/annotations/NotNull;() // invisible, parameter 0****
...
// access flags 0x19
public final static setTopPadding(Landroid/view/View;I)V
  @Lorg/jetbrains/annotations/NotNull;() // invisible, parameter 0
```

很明显，这里就是直接生成了两个静态函数`getTopPadding`和`setTopPadding`，并不是真的为View这个类添加了一个成员，那这个东西到底什么呢？我们把上面的代码稍微改一下：

```kotlin
var View.topPadding: Int = 0 
    inline get() = paddingTop
    set(value) = setPadding(paddingLeft, value, paddingRight, paddingBottom)
```

给这个成员加个默认值，可以看到，编辑器报错了
![](https://user-gold-cdn.xitu.io/2019/4/12/16a0fd7357d4602a?w=1322&h=124&f=png&s=38998)
并且告诉你这个属性没有幕后字段，所以不能初始化，好吧，官方给出了定义，**这就是一个属性（property）**。  

## 总结
在Kotlin中一个 property 不管有没有 backing field 都称之为 property，而在 Java 中 field + get、set方法一起才能是一个 property。  
如果我们从Java 的角度去看一个没有 backing field 的 property，可以理解为 Kotlin 对 以get、set开头这样的函数的语法糖，这种语法糖有什么用呢？个人觉得是为了DSL语法服务的，还是以上面那个topPadding为例，当你在用 DSL 语法设置一个view的时候，比如：

```kotlin
view.apply {
    background = getDrawable(R.drawable.bg)
    visibility = View.INVISIBLE
    ...
}
```
前面都是一个属性等于一个值，这个时候下面跟上 `topPadding = xxx`，语义十分清晰连贯，如果这里突然用一个 `setTopPadding(this,xxx)` ，不仅代码不美观，而且打断了阅读代码和编写代码的人的思维上的连贯性。

以上就是我对 Kotlin 的 Property 的理解