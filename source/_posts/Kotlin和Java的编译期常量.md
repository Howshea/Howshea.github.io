---
title: Kotlin 和 Java 的编译期常量
date: 2017-06-15 16:06:02
tags: 
  - Kotlin
  - Java
categories: 技术
---
在 Java 中，声明一个不可修改的常量用关键字 `final` 即可，例如：
```java
public final String HELLO_WORLD = "HelloWorld";
```
在 Kotlin 中，则是用 `val` 来表示常量：
```kotlin
val HELLO_WORLD = "HelloWorld"
```

但是两者有一些不同，在 Java 中声明了 `final`，则意味这这个常量变成了编译期常量，也就是说，编译过后的字节码，所有引用该常量的地方都会直接替换成值，这样有利于提高代码运行效率。
在 Kotlin 中，仅用 `val` 只意味着这是一个常量，而不是变成了编译器常量，如果想变成编译器常量，则需要额外加上 `const` 关键字：
```kotlin
const val HELLO_WORLD = "HelloWorld"
```
