title: Kotlin学习笔记
date: 2017.09.04 20:46:25
categories:
- 自学笔记
tags:
- Kotlin
---

> 项目未来可能需要使用`kotlin`开发，所以特此记录一下学习笔记，仅供参考，方便后期查询。已同步到`GitHub`上：[KotlinTest](https://github.com/xiaweizi/KotlinTest.git)

<!-- more -->

## Kotlin 简介

`kotlin` 的目标是成为一门全栈语言，主要有以下的特点：

- 已经成为`Android`的官方推荐语言
- 百分百的和`java`兼容,两者可以相互转换
- `JS`、`JVM`、`Native`多平台开发

## 数据类型

#### 1. 基本类型

    Boolean true/false
    Double 64
    Float  32
    Long   64
    Int    32
    Short  32
    Byte   8

    val aChar = '0'
    val bChar = '我'
    val cChar = '\u000f'
    
`Char`类型的转义字符

    \t          制表符
    \b          光标后退一个字符
    \n          回车
    \r          光标回到行首
    \'          单引号
    \"          双引号
    \\          反斜杠
    \$          美元符号，Kotlin 支持美元符号开头的字符串模板
    
#### 2. 基本类型的转换

不可隐式转换

    val anInt: Int = 5
    val aLong: Long = anInt.toLong()
    
必须得通过`.to类型`的方式进行数据的转换

**字符串**

- 一串`Char`
- 用双引号""引起来

    val aString: String = "Hello World!"
    
- 字符串比较

    a == b 表示比较内容 类似 Java 中的 equals
    a === b 表示比较对象是否相同

- 字符串模板

    println("hello, $name") -> "hello, 小明"

#### 3. Koltin 中的类和对象初始化

**类的定义**

- 类，一个抽象的概念
- 具有某些特征的事物的概括
- 不特定指代任何一个具体的事物

一般写法：

    /**
    * 其中类参数如果加上 var 修饰，那么他便是成员变量，反之则是普通的参数
    */
    class Student(var name: String, var age: Int){
        init {
            // ... 相当于构造函数中的代码
        }
    }

**对象**

- 是一个具体的概念，与类相对
- 描述某一个类的具体个体
- 举例：

    某些人、领导的车等等
    
**类和对象的关系**

- 一个类通常可以有很多歌具体的对象
- 一个对象本质上只能从属一个类
- 某一个人，他是工程师，但本质上还是属于人这一类

一般写法：

    val student: Student = Student("xiaweizi", 23)

**类的继承**

- 提取多个类的共性得到一个更为抽象的类，即父类
- 子类拥有父类的一切特征
- 子类也可以定义自己的特征
- 所有的类最终继承自`Any`,类似于`java`中的`Object`

#### 4. 空类型和智能转换

**空类型**

    // 定义
    val notNull: String = null // 错误，不可能为空
    val nullanle: String? = null // 正确，可以为空
    // 使用
    notNull.length // 正确，不可能为空所以可以直接使用
    nullable.length // 有可能为空，不能直接获取长度
    // 要想获取长度，可以通过以下两者方式
    nullable!!.length // 正确，强制认定 nullable 不可能为空，如果为空则会抛出空指针异常
    nullable?.length // 正确，若 nullable 为空，则返回 null
    
**智能类型转换**

    val child: Child = parent as Child // 类似于 Java 的类型转换，失败则抛出异常
    val child: Child = parent as? Child // 如果转换失败，返回 null

编译器智能识别转换：

    val parent: Parent = Child()
    if (parent is Child) {
        // parent 直接调用子类方法,不需要再进行强制转换
    }

    val string: String = null
    if (string != null) {
        // string.length 可以直接调用length 方法
    }

#### 5. 区间

一个数学上的概念，表示范围， `ClosedRange`的子类，`IntRange`最常用

基本用法：

    0..100 --> [0, 100]
    0 until 100 --> [0, 100)
    i in 0..100 表示 i 是否在区间[0, 100]中

#### 6. 数组

基本写法：

    val ints: IntArray = IntArrayOf(1,2,3,5)
    var charArray: CharArray = charArrayOf('a', 'b', 'c', 'd', 'e')
    var stringArray: Array<String> = arrayOf("aa", "bb", "cc", "dd", "e")
    
基本操作：

    print(charArray[index])
    ints[0] = 2
    ints.length
    cahrArray.joinToString("") // 讲 char 数组转换成字符串
    stringArray.slice(1..4) // 取出区间里的值

## 程序结构

#### 1. 常亮和变量

**常量**

    val a = 2
    类似 Java 中的 final
    不可被重复赋值
    运行时常量：val x = getX()
    编译期常量：const val x = 2
    
**变量**

    var a = 2
    a = 3 // 可以被再次赋值
    
**类型推导**

    val string = "Hello" // 推导出 String 类型
    val int = 5 // 推导出 Int 类型
    var x = getString() + 5 // String 类型

#### 2. 函数 Function

以特定功能组织起来的代码块

    // 最简单的打印信息，无返回的方法
    fun printMessage(message: String):Unit{
        println("$message")
    }
    // 拥有返回值得方法
    fun sum(first: Int, second: Int):Int {
        return first + second
    }
    // 可以简化成：
    fun sum(first: Int, second: Int) = first + second
    // 或者更简单的匿名函数
    val result = fun(first: Int, second: Int) = first + second

#### 3. Lambda 表达式

其实又是匿名函数

 **一般形式**：

    {传入参数 -> 函数体，最后一行是返回值}
    // 例如
    val sum = {first: Int, second: Int -> first + second}
    val printMessage = {message: String -> println(message)}

**类型标识**

    () -> Unit // 无参，返回值为 null
    (Int) -> Int // 传入整型，返回一个整型
    (String, (String) -> String) -> Boolean // 传入字符串、Lambda 表达式，返回Boolean

**Lambda 表达式的简化**

- 函数参数调用时最后一个`Lambda`可以移出去
- 函数参数只有一个`Lambda`，调用时小括号可以省略
- `Lambda`只有一个参数可默认为`it`
- 入参、返回值与形参一致的函数可以用函数引用方式作为实参传入

#### 4. 成员变量和成员方法

**成员变量的声明**

    // 第一种是在构造函数中声明
    class Student(var age: Int, name: String){
        // age 是成员变量 name 是局部变量
    }
    // 第二种是在函数体内声明
    var a = 0
        get() {
            field += 1
            return field
        }
        set(value) {
            println("set)
            field = value + 1
        }
    // 可以进行对 get 和 set 方法的重新定义
    
    // 属性的初始化尽量在构造方法中完成
    // var 用 lateinit 延迟初始化， val 用 lazy
    
    lateinit var sex: String
    val person: Person by lazy {
        Person()
    }

**成员方法**

在类中直接声明方法可以直接调用,包括`lambda`表达式

    // 方法的声明
    fun sum(a: Int, b: Int) = a + b
    val sum1 = {a: Int, b: Int -> a + b}
    // 方法的调用
    println(person.sum(1,2))
    println(person.sum1(3,5))

#### 5. 运算符

在`java`中运算符是不能重新定义重载的，只能按照原先的逻辑进行计算

而`Kotlin`则可以重新定义运算符，使用`operator`关键字，举了例子：

    // 定义一个用于计算复数的类
    class Complex(var real: Double, var imaginary: Double) {
        operator fun plus(other: Complex): Complex{
            return Complex(real+other.real, imaginary+other.imaginary)
        }
        
        // 重新 toString 方法
        overrride fun toString(): String {
            return "$real + ${imaginary}i"
        }
    }
    // 使用
    val complex1 = Complex(1, 2)
    val complex2 = Complex(2, 3)
    println(complex1 + complex2)
    // 输出结果为
    "3 + 5i"

关键就是这个方法，方法名必须是`plus`或者其他官方定义的运算符，参数有且仅有一个，类型自定义，返回值意识可以自定义的.

    operator fun plus(other: Complex): Complex{
            return Complex(real+other.real, imaginary+other.imaginary)
    }

#### 6. 表达式

**中缀表达式**

通过`infix`关键字修复方法，那么就可以不用通过 对象.方法() 的方式调用，而是直接 对象 方法名 参数的方式调用。举了例子

    class Student(var age: Int){
        infix fun big(student: Student): Boolean {
            return age > student.age
        }
    }
    // 如果没有 infix 的调用方式：
    println(Student(23).big(Student)(12))
    // 如果使用 infix 修饰的调用方式:
    println(Student(23) big Student(12))
    
**`if`表达式**

直接来个例子

    val a = 20
    val b = 30
    val flag: Int = if(a > b) a else b

**`When` 表达式**
 
 加强版的 `switch`，支持任意类型， 支持纯粹表达式条件分支(类似`if`)，举个栗子：
 
    val a = 5
    when(a) {
        is Int -> println("$a is Int")
        in 1..6 -> println("$a is in 1..6")
        !in 1..4 -> println("$a is not in 1..4")
        else -> {
            println("null")
        }
    }

 **`for`循环**
 
 基本写法
 
    for (element in elements)

**`while`循环**

基本写法

    while() {
    }
    do {
    } while()
跳过和终止循环

    跳过当前循环用 continue
    终止循环用 break

#### 6. 异常捕获

同样也是表达式，可以用来赋值，举个例子

    return try{
                x/y
            }
            catch(e: Exception) {
                0
            } finally {
                //...
            }

如果没有异常则返回`x/y`,否则返回`0`,`finally`中的代码无论如何还是要执行的。

#### 7. 具名参数、变长参数和默认参数

**具名参数：**给函数的实参附上形参

    fun sum(first: Int, second: Int) = first + second
    sum(second = 2, first = 1)

**变长参数：**用`varary`修饰，使用起来是和数组一样，某个参数可以接收多个值，可以不作为最后一个参数，如果传参时有歧义，需要使用具名参数。

    fun hello(vararg ints: Int, string: String) = ints.forEach(println(it))
    hello(1,3,4,5,string = "hello")
    // 如果最后一个参数也是 Int
    fun hello(varary ints: Int, anInt: Int)
    // 创建数组
    val arrayInt: IntArray = intArrayOf(1, 2, 3, 4)
    hello(ints = *arrayInt, anInt = 2)

**默认参数：**就是给参数传入一个默认的值

    fun hello(anInt: Int = 1, string: String)
    hello(string = "aaa")

## 面向对象

#### 1. 继承

**继承语法要点：**

- 父类需要`open`才可以被继承
- 父类方法、属性需要`open`才可以被覆写
- 接口、接口方法、抽象类默认为`open`
- 覆写父类(接口)成员需要`override`关键字

**语法要点：**

- `class A: B(), C, D`
- 继承类时实际上调用了父类的构造方法
- 类只能单继承，接口可以多实现

**接口代理：**

一个类可以直接将自己的任务委托给接口的方法实现，举个例子：

    interface Drive{
        fun drive()
    }
    
    interface Sing{
        fun sing()
    }
    
    class CarDrive: Drive{
        override fun drive() {
            println("我会开车呦")
        }
    }
    
    class LoveSing: Sing{
        override fun sing() {
            println("我会唱歌呦")
        }
    }
    
    class Manager(drive: Drive, sing: Sing): Drive by drive, Sing by sing
    
    fun main(args: Array<String>) {
        val carDrive = CarDrive()
        val loveSing = LoveSing()
        val manager = Manager(carDrive, loveSing)
        manager.drive()
        manager.sing()
    }

这样，`manager`不用做任何事情，完全交付给接口实现.

**接口方法冲突：**

接口方法可以有默认实现，通过`super<父类名>`.[方法名]([参数列表])

    interface A{
        fun a() = 0
    }
    
    interface B{
        fun a() = 1
    }
    
    interface C{
        fun a() = 2
    }
    
    class D(var aInt: Int): A,B,C{
        override fun a(): Int {
            return when(aInt){
                in 1..10 ->{
                    super<A>.a()
                }
                in 11..100 ->{
                     super<B>.a()
                 }
                else -> {
                    println("dd")
                    super<C>.a()
                }
            }
        }
    }

#### 2. 类及成员的可见性

跟`java`类似，`private、protected、public`，其中`internal`代表的是模块内可见

#### 3. `Object`

相当于`Java`中的单例模式，有以下特点

- 只有一个实例的类
- 不能自定义构造方法
- 可以实现接口、继承父类
- 本质上就是单例模式最基本的实现

        interface getDataSuccess{
            fun success()
        }
        
        abstract class getDataField{
            abstract fun failed()
        }
        
        object NetUtil: getDataField(), getDataSuccess{
            override fun success() {
                println("success")
            }
        
            override fun failed() {
                println("failed")
            }
        
            val state: Int = 0
            fun getData(): String = "请求成功"
        }

#### 3. 伴生对象和静态成员

相当于`java`中的静态方法

- 每个类可以对应一个伴生对象
- 伴生对象的成员全局独一份
- 如果`java`中想直接调用`kotlin`中的静态方法或者静态变量，可以考虑使用`JvmField JvmStatic`.

        open class Util private constructor(var anInt: Int) {
            companion object {
                @JvmStatic
                fun plus(first: Int, second: Int) = first + second
        
                fun copy(util: Util) = Util(util.anInt)
                @JvmField
                val tag = "tag"
            }
        }

#### 4. 方法的重载

通过给方法的参数配置默认值，即可实现方法的重载，按理说，一切可以拥有默认值的方法重载才是合理的方法重载。

名称形同、参数不同，跟返回值没有关系

    class OverLoadTest {
        @JvmOverLoads
        fun a(anInt: Int = 0, string: String="") = 1
    }
    
    val test = OverLoadTest()
    test.a(1, "")
    test.a()
    test.a(anInt = 2)
    test.a(string = "")

使用`JvmOverLoads`是为了方便`Java`中调用方法的重载.

#### 5. 扩展方法

`kotlin`中的扩展方法，我认为相当于`java`中的代理模式，拿到被代理的对象，然后进行一系列的操作。

    fun String.add(anInt: Int): String {
        var sb = StringBuilder()
        for (i in 0 until anInt) {
            sb.append(this)
        }
        return sb.toString()
    }
    
    operator fun String.times(anInt: Int): String {
        var sb = StringBuilder()
        for (i in 0 until anInt) {
            sb.append(this)
        }
        return sb.toString()
    }
    
    // 使用
    var string = "xiaweizi"
    println(string.add(5))
    println(string * (3))

#### 6. 属性代理

类似之前说的`var anInt: Int by lazy{2}`,懒赋值就是使用的属性代理，来看个例子：

    fun main(args: Array<String>) {
        val a: Int by DelegatesTest()
        println(a)
    
        var b: Int by DelegatesTest()
        b = 3
        println(b)
    }
    
    class DelegatesTest {
        private var anInt: Int? = null
        operator fun getValue(thisRef: Any?, property: KProperty<*>): Int {
            println("getValue")
            return anInt?:0
        }
    
        operator fun setValue(thisRef: Any?, property: KProperty<*>, value: Int): Unit {
            println("setValue")
            this.anInt = value
        }
    }

> `val` 对应 `getValue`，`var`对应`getValue和setValue`方法，这个时候声明的属性就全权交付给`DelegatesTest`类中的`anInt`代理，当`anInt`为空的时候返回`0`，否则返回`anInt`.

#### 7. JavaBean

使用`data`修饰类，类似`java`中的`javaBean`,默认实现了`set get toString`等方法，并拥有`componentN`方法.

不过有个缺点就是，无法被继承，没有无参构造函数，可以通过安装`allOpen`和`noArg`插件解决这个问题.

    data class UserBean(var name: String, var age: Int)
    
    val userBean: UserBean = UserBean("小芳", 23)
    println(userBean.name)
    println(userBean.toString())

    println(userBean.component1())
    println(userBean.component2())

    val (name, age) = userBean
    println("name: $name")
    println("age: $age")

> 至于这种写法`val (name, age) = userBean`，是因为定义了`component1`的运算符

    class Complex{
        operator fun component1() = "你好呀"
        operator fun component2() = 2
        operator fun component3() = 'a'
    }
    
    val complex = Complex()
    val (a, b, c) = complex
    println(a + b + c)

使用起来也是很简单的

#### 8. 内部类

- 定义在类内部的类
- 与类成员有相似的访问控制
- 默认是静态内部类，非静态用 `inner` 关键字
- `this@Outter` `this@Inner` 的用法
- 匿名内部类
    - 没有定义名字的内部类
    - 类名编译时生成，类似`Outter$1.class`
    - 可继承父类，实现多个接口，与`Java`注意区别

举个例子：

    class Outer{
        var string: String = "outer"
        class Inner1{
            var string: String = "inner1"
            fun sum(first: Int, second: Int) = first + second
        }
    
        inner class Inner2{
            var string: String = "inner2"
            fun cha(first: Int, second: Int) = first - second
            fun getInnerField() = this.string
            fun getOuterField() = this@Outer.string
        }
    }
    
    fun main(args: Array<String>) {
        val inner1 = Outer.Inner1()
        val inner2 = Outer().Inner2()
    
        println(inner1.sum(1, 2))
    
        println(inner2.cha(2, 1))
        println(inner2.getInnerField())
        println(inner2.getOuterField())
    }

**匿名内部类:**

    val listener: onClickListener = object : Father(), Mother, onClickListener{
        override fun sing() {
            println("mother sing")
        }

        override fun teach() {
            println("father teach")
        }

        override fun onClick() {
            println("匿名内部类")
        }
    }
    
> 使用`Object`实现匿名内部类

#### 9. 枚举和密封类

枚举是对象可数，每个状态相当于每个对象，是可以传构造参数的

密封类时子类可数，在`kotlin`大于1.1子类只需要与密封类在同一个文件加，保护子类的位置

    sealed class SealedClassTest{
        class sum(first: Int, seocnd: Int): SealedClassTest()
        class cha(first: Int, seocnd: Int): SealedClassTest()
    
        object Bean: SealedClassTest()
    }
    
    enum class HttpStatus(val anInt: Int){
        SUCCESS(0), FAILED(1), LOADING(2)
    }
    
    fun main(args: Array<String>) {
        val class1 = SealedClassTest.cha(1, 2)
        println(HttpStatus.SUCCESS)
    }
    
## 高阶函数

#### 1. 基本概念

- 传入或者返回函数的函数
- 函数引用 `::println`
- 带有`Receiver`的引用 `pdfPrinter::println`

有三种显示

    // 1. 包级函数
    intArray.forEach(::print)
    
    // 2. 类.方法
    intArray.forEach(Int::addOne)
    fun Int.addOne(): Unit {
        println("addOne:$this")
    }
    
    // 3. 对象.方法
    intArray.forEach(AddTwo()::addTwo)
    class AddTwo {
        fun addTwo(anInt: Int): Unit {
            println("addTwo:$anInt")
        }
    }

#### 2. 常用的高阶函数

常用的高阶函数还是有很多的，会简单的使用例子即可：

    // 遍历
    fun forEachTest() {
        val strings: Array<String> = arrayOf("aa", "ee", "bb", "ll")
    
        strings.forEach { println(it) } // 遍历每一个值
        strings.forEachIndexed { index, s -> println("index:$index,String:$s") } // 遍历 下标和值一一对应
    
    }
    
    // 重新拷贝一个值
    fun mapTest() {
        val strings: Array<String> = arrayOf("aa", "ee", "bb", "ll")
        var map = strings.map { "$it-test" }
        map.forEach { print("$it\t") }
    }
    
    // 将集合合体
    fun flatMapTest() {
        val lists = listOf(1..10,
                2..11,
                3..12)
    
        var flatMap = lists.flatMap {
            it.map {
                "No.$it"
            }
        }
        flatMap.forEach(::println)
    }
    
    fun reduceTest() {
        val ints = listOf(2, 3, 4, 5)
        println(ints.reduce { acc, i ->
            acc + i
        })
    }
    
    // 字符串连接
    fun foldTest(){
        val ints = listOf(2, 3, 4, 5)
        println(ints.fold(StringBuffer(), { acc, i -> acc.append("$i,") }))
        println(ints.joinToString(","))
    }
    
    fun filterTest() {
        val ints = listOf(1, 2, 3, 4, 5, 6)
        println(ints.filter { element -> element % 2 == 0 })
    }
    
    // 当值不是奇数就去，遇到偶数就停止了
    fun takeWhileTest() {
        val ints = listOf(1, 3, 3, 4, 5, 6)
        println(ints.takeWhile { it % 2 != 0 })
    }
    
    fun letTest() {
        findPerson()?.let { (name, age) -> println("name:$name, age:$age") }
        findPerson()?.apply { println("name:$name, age:$age") }
        with(findPerson()!!) { println("name:$name, age:$age") }
    }
    
    data class Person(val name: String, val age: Int)
    
    fun findPerson(): Person? {
        return Person("aa", 23)
    }

#### 3. 复合函数

有点类似数据中的`f(g(x))`

    fun main(args: Array<String>) {
        val add1 = {int: Int ->
            println("add1")
            int + 1}
        val add2 = {int : Int ->
            println("add2")
            int + 2}
        var add3 = add1 addThen (add2)
        println(add3(4))
    }
    
    
    infix fun <P1, P2, R> Function1<P1, P2>.addThen(function: Function1<P2, R>): Function1<P1, R> {
        return fun(p: P1): R{
            return function.invoke(this.invoke(p))
        }
    }
    
#### 4. Currying

简单来说就是多元函数变换成一元函数调用链式，举个简单的例子，这是优化之前：

    fun log(tag: String, out: OutputStream, message: String){
        out.write("[$tag], $message".toByteArray())
    }
    
优化之后

    fun log(tag: String)
        = fun(out: OutputStream)
        = fun(message: String)
        = out.write("[$tag], $message".toByteArray())

#### 5. 计算文件字符串个数的小例子

首先将字符串转换成字符串数组：

    val map: HashMap<Char, Int> = HashMap()
    var toCharArray = File("build.gradle").readText().toCharArray()
    
通过分组的方式，统计每个字符串的个数，并打印：

    toCharArray.groupBy { it }.map { it.key to  it.value.size }.forEach { println(it) }
    
## `kotlin`和`java`的混合开发

#### 1. 基本的交互操作

**属性读写**

- `Kotlin`自动识别 `Java Getter/Setter`
- `Java`操作`Kotlin`属性通过`Getter/Setter`

**空安全类型**

- `Kotlin`空安全类型的原理
- 平台类型`Platform Type`
- `Java`可以通过`@Nullable、@NotNull`

**几类函数的调用**

- 包级函数：静态方法
- 扩展方法：带`Receiver`的静态方法
- 运算符重载：带`Receiver`的对应名称的静态方法

**几个常用的注解**

- `@JvmField`：将属性编译为`Java变量`
- `@JvmStatic`：将对象的方法编译成功`Java`静态方法
- `@JvmOverloads`：默认参数生成重载方法
- `@JvmName`：制定`Kotlin`文件编译后的类名

**NoArg 和 AllOpen**

- `NoArg`为被标注的类生成无参构造
- `AllOpen`为被标注的类去掉`final`，允许被继承

**正则表达式**

- 用`Raw`字符串定义正则表达式
- `Java`的`Pattern`
- `Kotlin`的`Regex`

举个例子：

    val source = "Hello This my phone number: 010-12345678."
    val pattern = """.*(\d{3}-\d{8}).*"""

    Regex(pattern).findAll(source).toList().flatMap(MatchResult::groupValues).forEach(::print)
