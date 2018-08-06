---
title: Kotlin学习笔记（一）
date: 2017-08-19 20:36:01
tags: 
   - Android
   - Kotlin
categories: 学习
---
# Kotlin 简介
## Kotlin 是什么
- Android 官方开发语言，100% 兼容 Java
- Kotlin-Js 前端开发
- Kotlin-Jvm 服务端开发
- Kotlin-Native 本地执行程序
<!--more-->

## Hello World
```kotlin
fun main(args: Array<String>) {
    println("Hello World")
}
```
# 数据类型
## Boolean 类型
```kotlin
val aBoolean:Boolean=true
```
## Number 类型
1. Int ( 32 位)
	```kotlin
	//十六进制
	val hexInt: Int = 0xff
	//二进制
	val binInt: Int = 0b000011
	//kotlin 不支持直接表示 8 进制
	```
2. Long ( 64 位)
	```kotlin
	//未超过 Int 范围的数值需要加“L"
	val aLong:Long = 123L
	```
3. Float
	```kotlin
	//必须加 f
	val aFloat:Float=1.0F
	//1000.0
	val anotherFloat:Float=1E3F
	//负无穷
	val negativeInfinity: Float = Float.NEGATIVE_INFINITY
	//正无穷
	val positivieInfinity: Float = Float.POSITIVE_INFINITY
	//not a number
	val nan:Float = Float.NaN
	```
4. Double
	```kotlin
	val aDouble:Double = 2.0
	```
5. Short ( 16位 max : ( 2 ^ 15 ) - 1 == 32767 )
6. Byte ( 8位 max : 127)

> Kotlin 里不区分装箱和非装箱类型

## Char类型
- 占两个字节，表示一个 16 位的 Unicode 字符
- 用单引号
- 转义字符
	> \\t 制表符
	> \\b 光标后退一个字符
	> \\b 换行符
	> \\r 光标回到行首
	> \\' 单引号
	> \\" 双引号
	> \\\ 反斜杠
	> \\$ Kotlin支持$开头的字符串模板

## 基本类型转换
> Kotlin 不允许隐式转换

```kotlin
val anInt: Int = 5
val aLong: Long = anInt.toLong()
```
## 字符串
```kotlin
val string: String = "Hello"
val fromChars: String = String(charArrayOf('H', 'e', 'l', 'l', 'o'))
```
- Kotlin中字符串比较
  + 使用 `==` 等价于 Java 中的 `equals()`，表示比较内容
  + 使用 `===` 相当于 Java 的 `==` ，表示比较对象是否相同
- Kotlin支持字符串模板
	```kotlin
	val arg1 = 1
	val arg2 = 2
	println("$arg1 + $arg2 = ${arg1 + arg2}")
	```
- Kotlin 支持原生字符串，使用三个双引号隔开，可以包含没有转义的、换行符和任意其他字符,在原生字符串中不支持转义，但是仍然支持字符串模板
	```kotlin
	val rawString = """
	  \t
	  \n
	"""
	```

## 类
- 构造方法允许 **一个主构造方法** 和 **多个二级构造方法** ，每个二级构造函数需要委托给主构造函数，或者通过委托另一个二级构造方法来间接委托
	```kotlin
	class Girl constructor(){
	    constructor(character:String, appearance:String):this()
	    constructor(character: String, appearance: String, voice: String) : this(character, appearance)
	}
	```
- 只有一个构造方法时，`constructor()` 可以省略
	```kotlin
	class Girl(character: String, appearance: String, voice: String)
	```
- 不需要用new关键字
	```kotlin
	val aGirl:Girl = Girl("温柔","甜美","动人")
	```
- 可以直接把主构造中的参数直接声明成为类的属性，在参数名前加上 `var` 或 `val` ,在创建类的时候，调用构造函数就直接把它们进行了初始化，这样就不用在类中单独声明类的属性了
	```kotlin
	class Girl (var character: String, appearance: String, voice: String)
	//
	val aGirl:Girl = Girl("温柔","甜美","动人")
	//
	println(aGirl.character) //right
	println(aGirl.appearance) //wrong
	```
- `init{}` 相当于构造方法的方法体
	```kotlin
	class Girl (character: String){
	    init {
	        println("new a girl ,she is $character)
	    }
	}
	```

## 空类型安全
- 返回值可为空的方法或者值可为空变量必须加上 `?` ,否则编译无法通过
	```kotlin
	fun getName(): String? {
	    return null
	}
	```
- `?.` 代表如果不为空就返回后面的值
	```kotlin
	println(name?.length)
	```
- `?:` 代表如果为空就返回后面的值
	```kotlin
	val name = getName() ?: return
	```
- 如果能确定值确实不为空,可以用 `!!` 声明
	```kotlin
	val value: String? = "Hello World"
	println(value!!.length)
	```
## 智能类型转换
- 在已知为子类实例时，不需要像 Java 一样做强转，编译器会尽可能推导类型
	```kotlin
	//getName() 是 Child 类里的方法
	val parent: Parent = Child()
	if (parent is Child)
	    println(parent.name)
	```
- `as` 后面加上 `?` ,强制转换失败时返回空，而不会抛异常
	```kotlin
	val parent: Parent = Parent()
	val child: Child? = parent as? Child
	```
## 包
- 导入类可以指定别名
	```kotlin
	import net.println.kotlin.chapter2.市委.上海.市委书记 as 茶水大王
	//
	val 上海市委书记: 茶水大王 = 茶水大王("李")
	```
- 包和路径可以不一致，但是最好不要这样写

## 区间 ( Range )
- 接口 `ClosedRange` 的子类,  `IntRange` 最常用
- 用 `..` 表示闭区间，用 `until` 表示半开区间
	```kotlin
	val range:IntRange=0..1024 //[0,1024] ..操作符对应 rangeTo()方法
	//val range:IntRange= 0.rangeTo(1024)
	val range_exclusive:IntRange=0 until 1024 //[0,1024)
	```
- 用 `in` 或者 `contains()` 方法检查元素是否在区间内
	```kotlin
	//println(range.contains(40))
	println(40 in 0..100) //print true
	```
- 用 `in` 也可以迭代区间
	```kotlin
	for (i in range){
	    print("$i,")
	}
	```
## 数组
- 基本类型的数组使用 " 类型名 + ArrayOf " 系列方法构建，为了减少装箱和拆箱带来的开销
	```kotlin
	val arrayOfInt:IntArray = intArrayOf(1,3,5,7)
	val arrayofChar:CharArray = charArrayOf('H','e','l','l','o')
	```
- 其它类型使用 `arrayOf()`
	```kotlin
	val arrayOfString:Array<String> = arrayOf("我是","码农")
	```
- 基本用法
	```kotlin
	//return the array's length
	println(arrayOfInt.size)
	//foreach
	for (i in arrayOfInt){
	    println(i)
	}

	//use [index] instead of get and set
	println(arrayOfString[1])
	arrayOfString[1] = "程序员"

	//use some char insert a CharAarry and to be a String
	val s = arrayofChar.joinToString("") //return "Hello"

	//切片
	println(arrayOfInt.slice(1..3)) //return [3,5,7]
	```
# 程序结构
## 变量和常量
- `val` 表示常量, `var` 表示变量
	```kotlin
	var x = getX() //运行时常量
	const x = "..." //编译期常量
	```
## 函数
- 无返回值是返回 `Unit`, 等同于 Java 的 `void`, 返回类型是 `Unit` 时, 可以忽略不写
	```kotlin
	fun main(args: Array<String>){
	}
	fun sum(arg1: Int, arg2: Int): Int {
	    return arg1 + arg2
	}
	```
- 当函数体只是一个返回一个值的表达式时，可以简化写法：
	```kotlin
	//The return type can be omitted
	fun sum(arg1: Int, arg2: Int) = arg1 + arg2
	```
- 匿名函数，可以赋值给一个变量，除了没有名字，定义和使用和普通函数一样
	```kotlin
	//The return type can be omitted
	val intToLong = fun(x: Int) = x.toLong()
	//use
	println(intToLong(4))
	```
## Lambda 表达式
- Lambda 表达式其实也是函数的一种，写法是：{[参数列表]->[函数体，最后一行是返回值]}
	```kotlin
	val sum = { arg1: Int, arg2: Int -> arg1 + arg2 }
	val intToLong = { x: Int -> x.toLong() }
	```
- 无参：
	```kotlin
	val printlnHello = {
	    println("Hello")
	}
	```
- Lambda 表达式的返回值是最后一行的返回值
	```kotlin
	//return Int
	val sum = { arg1: Int, arg2: Int ->
	    println("$arg1 + $arg2 = ${arg1 + arg2}")
	    arg1 + arg2
	}
	```
- 变量名加 `()` 的方式是 `invoke()` 运算符
	```kotlin
	val sum = { arg1: Int, arg2: Int -> arg1 + arg2 }
	sum(1, 3)
	//等同于：
	sum.invoke(1, 3)
	```
- `forEach`
	```kotlin
	args.forEach ({ println(it)})
	//如果最后一个参数是 Lambda 表达式, 花括号可以移到小括号的外面：
	args.forEach() { println(it) }
	//小括号里什么的都没有，就可以删掉
	args.forEach { println(it) }
	//println 接收的参数和 forEach 接收的参数相同,引用
	args.forEach (::println)
	```
- 在 Lambda 表达式中 `return` 实际上不是中断这段表达式，而是中断它外部的函数
	```kotlin
	//"The End" 不会被打印，因为 forEach 表达式终止了 main 函数
	fun main(args: Array<String>) {
	    args.forEach {
	        if (it == "q")
	            return
	        println(it)
	    }
	    println("The End")
	}
	```
- 如果想在一个 Lamdba 表达式中中止当前这一次循环，可以为表达式加上标签，相当于 continue
	```kotlin
	//显式标签
	args.forEach foreach@ {
	    if (it=="q") return@foreach
	    println(it)
	}
	//隐式标签 
	args.forEach {
	    if (it == "q") return@forEach
	    println(it)
	}
	```
- 所有的函数的类型都是来自于 `Funtions.kt` 文件里的接口，从 `Function0` 到 `Function22`, 数字代表参数个数，所以 Kotlin 最多允许有22个参数的函数
	```kotlin
	/** A function that takes 0 arguments. */
	public interface Function0<out R> : Function<R> {
	    /** Invokes the function. */
	    public operator fun invoke(): R
	}
	......
	/** A function that takes 22 arguments. */
	public interface Function22<in P1, in P2, in P3, in P4, in P5, in P6, in P7, in P8, in P9, in P10, in P11, in P12, in P13, in P14, in P15, in P16, in P17, in P18, in P19, in P20, in P21, in P22, out R> : Function<R> {
	    /** Invokes the function with the specified arguments. */
	    public operator fun invoke(p1: P1, p2: P2, p3: P3, p4: P4, p5: P5, p6: P6, p7: P7, p8: P8, p9: P9, p10: P10, p11: P11, p12: P12, p13: P13, p14: P14, p15: P15, p16: P16, p17: P17, p18: P18, p19: P19, p20: P20, p21: P21, p22: P22): R
	}
	```
## 类成员 (成员方法、成员变量)
> 属性初始化尽量在构造器中完成
> 无法在构造方法中初始化的，尝试降级为局部变量
> var 用 lateinit 延迟初始化，val 用 lazy
> 可空类型谨慎用 null 直接初始化

- 方法就是类内部调用的函数，和函数只是叫法不同
- Kotlin 的 getter/setter 默认已实现, (val 常量没有 setter）
	```Kotlin,Java
	/* Kotlin 代码 */
	class A{
	    var b =0
	}
	/* 对应的 Java 代码 */
	public class A {
	    private int b=0;
	
	    public int getB() {
	        return b;
	    }
	
	    public void setB(int b) {
	        this.b = b;
	    }
	}
	```
- 可以复写 getter/setter，方法用 `field` 代替成员变量
	```kotlin
	class A{
	    var b = 0
	       get() {
	           println("some one tries to get b")
	           return field
	       }
	       set(value){
	           println("some one tries to set b")
	           field = value
	       }
	}
	```
- 设置 getter/setter 的访问权限，注意，get 方法的权限必须和该成员变量的访问权限保持一致,kotlin 中默认是 public 权限，不同于 Java 中默认是包访问权限
	```kotlin
	class A {
	    var b:Int = 0
	        private set
	}
	```
- Kotlin 允许成员变量不初始化，用 `lateinit` 即可，但是要注意使用时必须要初始化该成员，否则程序会崩溃
	```kotlin
	class A {
	    lateinit var c: String
	}
	...
	val a = A()
	a.c = "Hello"
	println(a.c)
	```
- 对于 val 类型的成员，可以使用委托懒加载，懒加载只有使用该成员时才会初始化
	```kotlin
	class X
	class A {
	    val e: X by lazy {
	        println("init X")
	        X()
	    }
	}
	```
## 基本运算符
> 运算符本质上是函数,通过对应的具名函数来定义

- 定义一个操作符
	```kotlin
	class Complex(var real:Double,var imageinary:Double){
	    operator fun plus(other:Complex):Complex{
	        return Complex(real+ other.real,imageinary + other.imageinary)
	    }
	    operator fun plus(other:Int):Complex{
	        return Complex(real+ other,imageinary)
	    }
	    override fun toString(): String {
	        return "$real + ${imageinary}i"
	    }
	}
	
	fun main(args: Array<String>) {
	    val c1 = Complex(3.0,4.0)
	    val c2 = Complex(2.0,7.6)
	
	    println(c1 + c2) //5.0 + 11.6i
	    println(c1 + 8) //11.0 + 4.0i
	}
	```
## 表达式
### 中缀表达式
> 只有一个参数，且用 `infix` 修饰的函数

- 示例
	```kotlin
	class Book{
	    infix fun on(desk:Desk):Boolean{
	        return false
	    }
	}
	class Desk
	...
	if (Book() on Desk()){  }
	```
### 分支表达式
> 返回值是每个分支的最后一句

- if else 表达式
	```kotlin
	private const val DEBUG = 1
	private const val USER = 0
	fun main(args: Array<String>) {
	    val mode = if (args.isNotEmpty() && args[0] == "1") DEBUG else USER
	}
	```
- when 表达式,加强版 switch
	```kotlin
	fun main(args: Array<String>) {
	    val x = 99
	    //条件语句
	    when (x) {
	        102 -> println("right! $x")
	        in 1..100 -> println("$x is in 1-100")
	        !in 1..100 -> println("$x is not in 1-100")
	    }
	    //无自变量的 when，可以检查在 when 条件左边想要的任何事
	    val mode =when{
	        args.isNotEmpty() && args[0]=="1"-> 1
	        else -> 0
	    }
	    println(mode)
	}
	```
## 循环语句
- for 循环
	```kotlin
	for(arg in args){
	    println(arg)
	}
	// args.withIndex() 返回一个迭代器
	for(indexedValue in args.withIndex()){
	    println("${indexedValue.value} -> ${indexedValue.index}")
	}
	//可以写成两个参数，是因为Data Class
	for((index,value) in args.withIndex()){
	    println("$index -> $value")
	}
	```
- while, do while 语句，与 Java 基本相同，break 跳出，continue 跳过
- 多层循环嵌套的终止结合标签使用
	```kotlin
	Outter@for(...) {
	    Inner@while(i<0) { if(...) break@Outter }
	}
	```
## 异常捕获 (try, catch, finally)
- try catch 语句也是表达式，可以有返回值.finally先执行，然后再返回结果
	```kotlin
	val result = try {
	    args[0].toInt()/args[1].toInt()  // 1 / 0
	}catch (e:Exception){
	    e.printStackTrace()
	    0
	}finally {
	    println("finish")
	}
	println(result)  // print 0
	```
## 参数
### 具名参数
> 给函数的实参附上形参

```kotlin
fun sum(arg1: Int, arg2: Int) = arg1 + arg2
sum(arg1 = 2, arg2 = 3)
```
### 变长参数
- 变长参数可以当数组一样使用
	```kotlin
	fun main(vararg args: String) {
	   for (arg in args){
	   }
	}
	```
- 变长参数在 kotlin 中可以不放在最后一位，但是变长参数之后的参数需要使用具名参数
	```kotlin
	fun hello(double: Double,vararg ints:Int,string: String){
	    println(double)
	    ints.forEach(::print)
	    println("\n$string")
	}
	// * 运算符用于变长参数展开数组，目前只支持数组且不能重载
	val array = intArrayOf(1,3,5,7,9)
	hello(3.0,*array,string = "kotlin")
	```
- 有歧义时要用具名参数指定

# 面向对象
## 接口和抽象类
- 接口内变量不能初始化,而抽象类内的变量必须初始化
	```kotlin
	abstract class A {
	    var i = 0
	}
	interfac B{
	    //相当于生成了一个 getJ 和 setJ 方法
	    var j: Int
	}
	```
- 不同于 Java，kotlin 的接口的方法可以有默认实现,如果有默认实现，实现类可以不用实现该方法
	```kotlin
	interface B {
	    var j:Int
	    fun hello(){
	        println(j)
	    }
	}
	```
## 继承
- 类和成员默认都是 `public` 和 `final` ，变为可被继承需要用 `open` 声明，或者是 `abstract`
- 继承的成员需要使用 `override` 关键字
- `by` 接口代理，有点曲线救国地支持多继承的意思
	```kotlin
	class SeniorManager(val driver: Driver,val writer: Writer):Driver by driver,Writer by writer
	class CarDriver:Driver{
	    override fun drive() {
	        println("开车")
	    }
	}
	class PPTWriter:Writer{
	    override fun write() {
	        println("写PPT")
	    }
	}
	interface Driver{
	    fun drive()
	}
	interface Writer{
	    fun write()
	}
	fun main(args: Array<String>) {
	    val driver = CarDriver()
	    val writer = PPTWriter()
	    val seniorManager = SeniorManager(driver,writer)
	    seniorManager.drive()
	    seniorManager.write()
	}
	```
- 继承方法名重复问题，可以用 super 加模板的方式调用解决，但是方法的返回类型必须相同
	```kotlin
	interface B {
	    fun x(): Int = 1
	}
	interface C {
	    fun x(): Int = 0
	}
	class D : B, C {
	    var y: Int = 0
	    override fun x(): Int {
	        return when {
	            y > 0 -> y
	            y < -100 -> super<B>.x()
	            else -> super<C>.x()
	        }
	    }
	}
	```
## 类及其成员的可见性
- `private, protected,internal,public`
- `internal` 是同一 module 内可见

## object
- 和 class 其它都一样，只是只有一个实例，相当于直接实现了单例
- 在 java 代码中调用 kotlin 的 object 类：
	```Kotlin
	//Kotlin 代码
	object Player{
	    fun printName(): String {
	        return "Player"
	    }
	}
	//Java 代码
	Player.INSTANCE.printName();
	```
## 伴生对象与静态成员
- 静态成员首先考虑使用包级函数、变量替代
- 伴生对象内的方法和变量，相当于 Java 中的静态方法和静态变量
	```kotlin
	class Latitude private constructor(val value: Double) {
	    companion object {
	        fun ofDouble(double: Double): Latitude {
	            return Latitude(double)
	        }
	        val TAG: String = "Latitude"
	    }
	}
	fun main(args: Array<String>) {
	    Latitude.ofDouble(1.1)
	}
	```
- 如果要在 Java 中一样调用，需要为伴生对象中的方法加上 `@JvmStatic` 注解，变量上加 `@JvmField` ，不加的话，调用方式略有区别：
	```java
	//不加注解
	Latitude.Companion.ofDouble(1.1);
	System.out.println(Latitude.Companion.getTAG());
	//加注解
	Latitude.ofDouble(1.1);
	System.out.println(Latitude.TAG);
	```

## 方法重载
- 方法签名包含方法名和方法参数列表，不包括返回类型
- 方法重载与方法的返回类型无关


## 默认参数
- 默认参数的存在可以代替方法重载，一个最佳实践是，如果一个方法重载不能用默认参数代替，那这个重载很可能有隐藏的问题，所以避免定义关系不大的重载
- 在 Java 中调用 Kotlin 里的带默认参数的方法，本身不支持默认参数，需要在方法上加上 `@JvmOverloads` 注解

## 扩展成员
- 为类添加扩展方法，避免写过多的工具类
	```kotlin
	fun main(args: Array<String>) {
	    print("abc".multiply(3))
	    //print abcabcabc
	}
	
	fun String.multiply(int: Int):String{
	    val stringBuilder = StringBuilder();
	    for (i in 1..int){
	        stringBuilder.append(this)
	    }
	    return stringBuilder.toString()
	}
	```
- 操作符扩展
	```kotlin
	fun main(args: Array<String>) {
	    println("abc" * 4) // print abcabcabcabc
	}
	operator fun String.times(int: Int):String{
	    val stringBuilder = StringBuilder()
	    for (i in 0 until int){
	        stringBuilder.append(this)
	    }
	    return stringBuilder.toString()
	}
	```

	```java
	//扩展方法在java中调用
	System.out.println(ExtendsKt.times("abc", 3));
	```
- 扩展属性, 注意扩展属性不能初始化，类似接口属性
	```kotlin
	fun main(args: Array<String>) {
	    println("abc".a)
	    println("abc".b)
	}
	val String.a: String
	    get() = "abc"
	var String.b: Int
	    set(value) { }
	    get() = 5
	```
## 属性代理
- 定义方法：
	`val/var <property name>: <Type> by <expression>`
- 代理者需要实现相应的 setValue/getValue 方法
	```kotlin
	class Delegates{
	    var hello by X()
	}
	class X{
	    private var value:String?=null
	    operator fun getValue(thisRef:Any?,property:KProperty<*>):String {
	        return value?:""
	    }
	    operator fun setValue(thisRef: Any?,property: KProperty<*>,value:String){
	        println("set ${property.name}: $value")
	        this.value = value
	    }
	}
	fun main(args: Array<String>) {
	    val delegates = Delegates()
	    delegates.hello = "hello" //print set hello: hello
	}
	```
## 数据类
- dataclass 默认实现了 toString(),hashCode() 等方法
	```kotlin
	data class Country(val id:Int,val name:String)
	
	fun main(args: Array<String>) {
	    val china = Country(0,"China")
	    println(china)
	    println("${china.component1()},${china.component2()}")
	    val(id,name) = china
	    println("$id,$name")
	    /* print
	    Country(id=0, name=China)
	    0,China
	    0,China
	    */
	}
	```
- 关于 component,不是dataclass独有的,普通的类也可以使用
	```kotlin
	class ComponentX {
	    operator fun component1(): String {
	        return "Hello! we are "
	    }
	    operator fun component2(): Int {
	        return 1
	    }
	    operator fun component3(): Int {
	        return 1
	    }
	    operator fun component4(): Int {
	        return 0
	    }
	}
	fun main(args: Array<String>) {
	    val componentX= ComponentX()
	    val(a,b,c,d) = componentX
	    println("$a$b$c$d")
	    /* print
	    Hello! we are 110
	     */
	}
	```
- dataclass 没有无参构造器，也不能继承，可以使用 allOpen 和 noArg 插件来去除 final 和添加无参构造方法

## 内部类
- kotlin 中内部类默认是静态内部类，这与 Java 不同，如果要变成非静态，需要加上 inner 关键字
	```kotlin
	class Outter{
	    inner class Inner
	    class Inner1
	}
	
	fun main(args: Array<String>) {
	    val inner = Outter().Inner()
	    val inner1 = Outter.Inner1()
	}
	```
- 非静态内部类引用外部类成员，使用 `this@<class name>.<property name>` 
	```kotlin
	class Outter{
	    val a:Int=5
	    inner class Inner{
	        fun hello(){
	            println(this@Outter.a)
	        }
	    }
	}
	```
- kotlin 的匿名内部类允许继承和多实现，Java 里不可以
	```kotlin
	class Outter
	interface OnClickListener {
	    fun onClick()
	}
	class View {
	    var onClickListener: OnClickListener? = null
	}
	fun main(args: Array<String>) {
	    val view = View()
	    //这里的匿名内部类同时继承了 Outter 类
	    view.onClickListener = object: Outter(),OnClickListener{
	        override fun onClick() {}
	    }
	}
	```
- 匿名内部类实际上是有名字的

## 枚举
- kotlin 的枚举类允许添加构造方法和成员方法，但是枚举和其它成员要求用分号隔开
	```kotlin
	enum class LogLevel(val id:Int){
	    VERBOSE(1),DEBUG(2),INFO(3),WARN(4),ERROR(5),ASSET(6);
	    override fun toString(): String {
	        return "$name, $ordinal"
	    }
	}
	fun main(args: Array<String>) {
	    //打印 DEBUG 的索引
	    println(LogLevel.DEBUG.ordinal)
	    //遍历打印所有的枚举
	    LogLevel.values().map(::println)
	    //获取实例
	    LogLevel.valueOf("WARN")
	}
	```
## 密封类
- 子类可数的类，kotlin1.1之前其子类必须是内部类，1.1之后允许子类定义在类的外部，但是必须在同一文件里
	```kotlin
	sealed class PlayerCmd
	class Play(val url: String, val position: Long = 0) : PlayerCmd()
	class Seek(val position: Long) : PlayerCmd()
	object Pause : PlayerCmd()
	object Resume : PlayerCmd()
	object Stop : PlayerCmd()
	```
- 与枚举不同的是：枚举类的每个枚举常量只存在一个实例，而密封类的一个子类可以有可包含状态的多个实例
