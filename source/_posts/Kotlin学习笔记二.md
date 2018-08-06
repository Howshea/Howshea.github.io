---
title: Kotlin学习笔记（二）
date: 2017-09-11 17:31:53
tags: 
   - Android
   - Kotlin
categories: 学习
---
> 这一篇是高阶一点的知识

<!--more-->
# 高阶函数
## 基本概念
- 传入或返回函数的函数
- 包级函数引用:
	```kotlin
	fun main(args: Array<String>) {
	    args.forEach(::println)
	    args.forEach(::printTwoLn)
	}
	fun printTwoLn(message: Any?) {
	    System.out.println("$message\n")
	}
	```
- 类名加函数名应用，包含一个隐藏的参数——调用者本身
	```kotlin
	fun main(args: Array<String>) {
	    args.filter(String::isNotEmpty)
	    println(args.filter(String::isOops))// print [!,!]
	}
	fun String.isOops(): Boolean {
	    return equals("!")
	}
	```
- 带有 Receiver 的引用（从 Kotlin1.1 之后才支持的语法）
	```kotlin
	fun main(args: Array<String>) {
	    val pdfPrinter = PdfPrinter()
	    args.forEach(pdfPrinter::println)
	}
	class PdfPrinter {
	    fun println(any: Any) {
	        kotlin.io.println(any)
	    }
	}
	```
## 常见高阶函数
- `map` 映射，返回类型任意
	```kotlin
	val list = listOf(1, 3, 4, 6, 23, 53)
	val newList = list.map { it * 2 + 3 }
	println(newList) // print [5, 9, 11, 15, 49, 109]

    val newList2 = list.map(Int::toDouble)
    println(newList2) //print [1.0, 3.0, 4.0, 6.0, 23.0, 53.0]
	```
- `flatMap` 铺平集合的集合,返回任意类型
	```kotlin
	var listFlat = listOf<IntRange>(
	        1..3,
	        8..10
	)
	val flatMapList = listFlat.flatMap {
	    it.map {
	        "No. $it"
	    }
	}
	//print [No. 1, No. 2, No. 3, No. 8, No. 9, No. 10]
	println(flatMapList)
	```
- `reduce` 表达式包含两个参数，上一次运算的结果和当前的迭代的值，acc 必须是 i 的父类
	```kotlin
	//求和
	val sum = list.reduce { acc, i -> acc + i }
	//阶乘
	val factorial = (1..6).reduce { acc, i -> acc * i }
	```
- `fold` ，reduce 的增强版，增加一个初始值的参数，Lambda 表达式参数的类型没有限制，`foldRight` 是 `fold` 倒序操作，还有 `foldIndexed`, `foldRightIndexed` 提供更多功能
	```kotlin
	//return "List,1,3,4,6,23,53"
	list.fold("List") { acc, i -> "$acc,$i" }
	```
- `filter` , 用于过滤，还有 `filterIndexed` 等增强型函数
- `takeWhile` 和 filter 类似，但是遇到第一个不符合条件的就停止迭代
	```kotlin
	//遇到第一个偶数就停止迭代
	list.takeWhile { it % 2 == 1 }
	```
- `let`
	```kotlin
	data class Person(val name: String, val age: Int) {
	    fun work() {}
	}
	fun findPerson(): Person? {
	    return null
	}
	fun main(args: Array<String>) {
	    findPerson()?.let {
	        it.work()
	        println(it.age)
	    }
	}
	```
- `apply`，和 `let` 略有不同
	```kotlin
	fun main(args: Array<String>) {
	    findPerson()?.apply { 
	        work()
	        println(age)
	    }
	}
	```
- `with`， 和 `apply` 类似，但是不能是可空类型
	```kotlin
	val br = BufferedReader(FileReader("hello.txt"))
	    with(br){
	        var line:String?
	        while (true){
	            line =readLine()?:break
	            println(line)
	        }
	        close()
	    }
	```
- `use` , 避免写 close,it不能省略
	```kotlin
	BufferedReader(FileReader("hello.txt")).use {
	    var line: String?
	    while (true) {
	        line = it.readLine() ?: break
	        println(line)
	    }
	}
	```
## 尾递归优化
- 对于尾递归的函数，前面加上 `tailrec` 关键字
	```kotlin
	data class ListNode(val value: Int, var next: ListNode?=null)
	tailrec fun findListNode(head: ListNode?, value: Int): ListNode? {
	    head ?: return null
	    if (head.value == value) return head
	    return findListNode(head.next, value)
	}
	```
## 闭包
- 函数运行的环境，持有函数运行状态，函数内部可以定义函数，也可以定义类
- 例子，斐波那契数列
	```kotlin
	fun fibonacci(): Iterable<Long> {
	    var first = 0L
	    var second = 1L
	    return Iterable {
	        object : LongIterator() {
	            override fun nextLong(): Long {
	                val result = second
	                second += first
	                first = second - first
	                return result
	            }
	            override fun hasNext() = true
	        }
	
	    }
	}
	fun main(args: Array<String>) {
	    fibonacci()
	            .takeWhile { it <= 100 }
	            .forEach(::println)
	}
	```
## 函数复合
- f(g(x))
	```kotlin
	val add5 = { i: Int -> i + 5 }
	val multiplyBy2 = { i: Int -> i * 2 }
	infix fun <P1, P2, R> Function1<P1, P2>.andThen(function: Function1<P2, R>): Function1<P1, R> {
	    return fun(p1: P1): R {
	        return function(this(p1))
	    }
	}
	fun main(args: Array<String>) {
	    val add5AndMultiPlyBy2 = add5 andThen multiplyBy2
	    println(add5AndMultiPlyBy2(11)) //print 32
	}
	```
## 科里化（Currying）
- 多元函数变换成一元函数调用链
	```kotlin
	//原始函数
	fun log(tag: String, target: OutputStream, message: Any?) {
	    target.write("[$tag] $message\n".toByteArray())
	}
	//方式一
	fun log(tag: String)
	        = fun(target: OutputStream)
	        = fun(message: Any?)
	                = target.write("[$tag] $message\n".toByteArray())
	//调用
	log("benny")(System.out)("Hello")
	//方式二（扩展函数）
	fun <P1, P2, P3, R> Function3<P1, P2, P3, R>.curried()
	        = fun(p1: P1) = fun(p2: P2) = fun(p3: P3) = this(p1, p2, p3)
	//调用
	::log.curried()("benny")(System.out)("Hello")
	```
## 偏函数
- 传入部分参数得到的新函数
- 示例1,需要固定的参数在前面
	```kotlin
	val consoleLogWithTag = (::log.curried())("benny")(System.out)
	consoleLogWithTag("Hello Again.")
	```
-  示例2，需要固定的参数在后面
	```kotlin
	fun makeString(byteArray: ByteArray, charset: Charset) = String(byteArray, charset)
	fun <P1, P2, R> Function2<P1, P2, R>.partial2(p2: P2) = fun(p1: P1) = this(p1, p2)
	fun main(args: Array<String>) {
	    val makeStringFromGbkBytes = ::makeString.partial2(charset("GBK"))
	    makeStringFromGbkBytes("World".toByteArray())
	}
	```
# 领域特定语言 DSL
## HTML DSL
- 示例：
	```kotlin
	html {
	    "id"("htmlId")
	    head {
	        "id"("headId")
	    }
	    body {
	        id = "bodyId"
	        `class` = "bodyCl
	        "a"{
	            "href"("https
	            +"kotlin博大精深"
	        }
	    }
	}.render().let(::println)
	```
## Gradle Kotlin 脚本
- 把 build.gradle 文件改成 build.gradle.kts
- 修改为 Kotlin 的 DSL
	```kotlin
	group = "com.howshea.KotlinLearning2"
	version = "1.0-SNAPSHOT"
	buildscript {
	    extra["kotlin_version"] = "1.1.4-2"
	    repositories {
	        mavenCentral()
	    }
	    dependencies {
	        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:${extra["kotlin_version"]}")
	    }
	}
	apply {
	    plugin("java")
	    plugin("kotlin")
	}
	configure<JavaPluginConvention> {
	    setSourceCompatibility(1.8)
	}
	repositories {
	    mavenCentral()
	}
	dependencies {
	    "compile"("org.jetbrains.kotlin:kotlin-stdlib-jre8:${extra["kotlin_version"]}")
	    "TestCompile"("junit:junit:4.12")
	}
	```
# 协程 Coroutine
## 协程的概念
- 协作程序，解决异步问题
- 应用层完成调度
- 支持协程的语言：Lua, C# 等

## 协程要解决的问题
- 异步代码像同步代码一样直观
- 简化异步代码处理
- 轻量级的并发方案

## Kotlin 的协程支持
- 编译器对 suspend 函数的编译支持
- 标准库 API 的支持
- kotlinx.coroutine 应用级支持

## 协程的基本 API
- `createCoroutine` 创建协程
- `startCoroutine` 启动协程
- `suspendCoroutine` 挂起协程
- `Continuation` 接口，运行控制类，负责结果和异常的返回
- `CoroutineContext` 接口，运行上下文，资源持有，运行调度
- `ContinuationInterceptor` 接口，协程控制拦截器，可用来处理协程调度

## 原理剖析

### 执行流程
1. 协程编译成状态机
2. `suspend` 函数即状态转移

### 运行结果
1. 正常结果通过 resume 返回
2. 异常通过 resumeWithException 抛出

## buildSequence 序列生成器
```kotlin
fun main(args: Array<String>) {
    for (i in fibonacci){
        println(i)
        if (i> 100) break
    }
}
val fibonacci = buildSequence{
    yield(1)
    var cur = 1
    var next = 1
    while (true){
        yield(next)
        val tmp = cur + next
        cur = next
        next = tmp
    }
}
```

## Kotlinx.coroutine 框架
- 主要模块
  + `kotlinx-coroutines-core` 核心库
  + `kotlinx-coroutines-jdk8` Java8 支持库
  + `kotlinx-coroutines-nio` 异步IO库
  + `kotlinx-coroutines-reactive` Reactive Streams 支持
  + `kotlinx-coroutines-reactor` Reactor 支持
  + `kotlinx-coroutines-rx1` RxJava 1.x 支持
  + `kotlinx-coroutines-rx2` Rxjava2.x 支持
  + `kotlinx-coroutines-android` Android UI 支持
  + `kotlinx-coroutines-javafx` JavaFx UI 支持
  + `kotlinx-corountines-swing` Swing UI 支持

# Kotlin 与 Java 混合开发

## 基本操作

### 属性读写
- Kotlin 自动识别 Java Getter/Setter
- Java 操作 Kotlin 属性通过 Getter/Setter

### 空安全类型
- Java 不支持空安全，需要开发者自己注意
- 在 Java 代码中使用 `@Nullable` 和 `@NotNull` 

### 几类函数的调用
- 包级函数：静态方法
- 扩展方法：带 Receiver 的静态方法
- 运算符重载：带 Receiver 的对应名称的静态方法

### 常用注解的使用
- `@JvmField`：将属性编译为 Java 变量
- `@JvmStatic`：将对象的方法编译成 Java 静态方法
- `@JvmOverloads`：默认参数生成重载方法
- `@file` : JvmName : 指定 Kotlin 文件编译后的类名

### NoArg 与 AllOpen
- NoArg 为被标注的类生成无参构造
- AllOpen 为被标注的类去掉 final，允许被继承

### 泛型
- 通配符 Kotlin 的 * 对应于 Java 的 ？
- 协变和逆变 out/in 对应 Java 的 <? extends E> 和 <? super E>
- kotlin 没有 Raw 类型， Java 的 List 对应 Kotlin 的 List<*>

## SAM 转换
- Single Abstract Method
- SAM 转换的条件
  + Java 的接口，单一接口方法
- 注意转换后实例的变化

## Kotlin 的正则
```kotlin
val source = "Hello , This my phone number: 010-12345678."
//使用raw string，不需要转义字符
val pattern = """.*(\d{3}-\d{8}).*"""
Regex(pattern).findAll(source).toList().flatMap(MatchResult::groupValues).forEach(::println)
```
