---
title: 『Thinking in Java 读书笔记』—— 13-字符串
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_13
date: 1994-04-15 15:13:33

---

[Thinking in java 读书笔记](http://xiaweizi.cn/categories/Thinking-In-Java-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)

## 不可变 String

> `String` 对象是不可变的，具有只读特性。

```Java
public class Immutable {  
    public static String upcase(String s) {  
        return s.toUpperCase();  
    }  
  
    public static void main(String[] args) {  
        String q = "howdy";  
        print(q); // howdy  
        String qq = upcase(q);  
        print(qq); // HOWDY  
        print(q); // howdy(原有 String 没有改变)  
    }  
}  
// out:
// howdy  
// HOWDY  
// howdy 
```

<!-- more -->

## 重载运算符 + 与 StringBuilder

#### 重载

>  一个操作符在应用于特定的类时， 被赋予特殊意义。用于`String` 的 + 和 +=  是java中仅有的两个重载过的操作符，而java 并不允许程序员重载任何操作符

```Java
public class Concatenation {  
    public static void main(String[] args) {  
        String mango = "mango";  
        String s = "abc" + mango + "def" + 47;  
        System.out.println(s);  
    }  
}  
// out:
// abcmangodef47
```

字符串连接符 + 的性能非常低下。。因为为了生成最终的 `string`， 会产生大量需要垃圾回收的中间对象.

#### 通过 javap 来反编译 Concatenation

```c++
Compiled from "Concatenation.java"  
public class chapter13.Concatenation {  
  public chapter13.Concatenation();  
    Code:  
       0: aload_0  
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V  
       4: return  
  
  public static void main(java.lang.String[]);  
    Code:  
       0: ldc           #2                  // String mango  
       2: astore_1  
       3: new           #3                  // class java/lang/StringBuilder  
       6: dup  
       7: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V  
      10: ldc           #5                  // String abc  
      12: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;  
      15: aload_1  
      16: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;  
      19: ldc           #7                  // String def  
      21: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;  
      24: bipush        47  
      26: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;  
      29: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;  
      32: astore_2  
      33: getstatic     #10                 // Field java/lang/System.out:Ljava/io/PrintStream;  
      36: aload_2  
      37: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V  
      40: return  
}  
```

编译器自动引入了 `java.lang.StringBuilder` 类，即使源代码中没有使用 `StringBuilder`， 但是显然`StringBuilder` 更加有效。

#### 编译器对 String 的优化

```Java
// 利用 StringBuilder.append() 来重载 + 运算符  
public class WhitherStringBuilder {  
    public String implicit(String[] fields) { // 方法一：使用多个String对象  
        String result = "";  
        for (int i = 0; i < fields.length; i++) // （效率低）隐式创建 StringBuilder  
            result += fields[i];   
        return result;  
    } // 因为 StringBuilder是在循环内创建的，这意味着 每经过循环一次，就会创建一个新的 StringBuilder对象  
  
  
    public String explicit(String[] fields) {  // 方法二：使用StringBuilder，因为效率高  
        StringBuilder result = new StringBuilder(); // （效率高）显式创建 StringBuilder  
        for (int i = 0; i < fields.length; i++)  
            result.append(fields[i]);  
        return result.toString();  
    }  
}
```

#### StringBuilder 其他特点

可以为`StringBuilder` 预先指定大小，如果知道最终的字符串长度，可以预先指定`StringBuilder`的大小， 以避免多次 重新分配缓冲。

如果要在toString() 方法中使用循环的话，最好自己创建一个`StringBuidler` 对象。

 insert, replace, substring, reverse, 最常用的方法是 `append` 和 `toString()` 方法。

`StringBuilder` 线程不安全，效率高， `StringBuffer` 线程安全，效率低。

## 无意识的递归

**所有java的根基类都是 Object，** 所以容器类都有 `toString()` 方法。 容器的`toString()` 方法都能够表达容器自身和容器所包含的 对象。

```Java
// 无限递归 使得 java虚拟机栈被顶满, 然后抛出异常  
public class InfiniteRecursion {  
    @Override  
    public String toString() {  
         // toString() 中的this关键字是 引起无限递归的原因  
//      return " InfiniteRecursion address: " + this + "\n"; // Exception in thread "main" java.lang.StackOverflowError  
        return " InfiniteRecursion address: " + super.toString() + "\n";  
    }  
  
    public static void main(String[] args) {  
        List<InfiniteRecursion> v = new ArrayList<InfiniteRecursion>();  
          
        for (int i = 0; i < 10; i++)  
            v.add(new InfiniteRecursion());  
        System.out.println(v);  
    }  
}    
/* 
[ InfiniteRecursion address: chapter13.InfiniteRecursion@15db9742 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@6d06d69c 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@7852e922 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@4e25154f 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@70dea4e 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@5c647e05 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@33909752 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@55f96302 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@3d4eac69 
,  InfiniteRecursion address: chapter13.InfiniteRecursion@42a57993 
] 
*/
```

**这里发生了自动类型转换：** 由 `InfiniteRecursion`类型转换为 `String` 类型。 this前面的是字符串，后面是换行符， 所以 `this` 转换为 `String`， 即调用了 `this.toString()` 方法， 于是就发生了 **递归调用 toString() 方法**，无限递归使得 java 虚拟机栈被顶满； 然后抛出异常；**把this换做 super.toString() 方法后 执行成功**

## String 上的操作

`String` 对象的基本方法

![String基本方法1](http://owj4ejy7m.bkt.clouddn.com/2018-04-15-String%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%96%B9%E6%B3%95.png)

![String基本方法2](http://owj4ejy7m.bkt.clouddn.com/2018-04-15-String%E5%9F%BA%E6%9C%AC%E6%96%B9%E6%B3%95.png)

当需要改变字符串的内容， `String` 类的方法都会返回一个新的 `String` 对象，如果没有改变，则返回原对象的引用。

## 格式化输出

通过 `System.out.format()` 输出格式。

```java

// System.out.format() 输出格式  
public class SimpleFormat {  
    public static void main(String[] args) {  
        int x = 5;  
        double y = 5.332542;  
        // The old way:  
        System.out.println("Row 1: [" + x + " " + y + "]");  
        // The new way:  
        System.out.format("Row 1: [%d %f]\n", x, y); // format() 方法的荔枝  
        // or  
        System.out.printf("Row 1: [%d %f]\n", x, y); // printf() 方法荔枝  
    }  
}    
/* 
Row 1: [5 5.332542] 
Row 1: [5 5.332542] 
Row 1: [5 5.332542] 
*/ 
```

具体转换字符格式

| 字符 | 含义                      |
| ---- | ------------------------- |
| d    | 整数型(10进制)            |
| c    | Unicode 字符              |
| b    | Boolean 值                |
| s    | String                    |
| f    | 浮点数                    |
| x    | 整数(16进制)              |
| h    | 散列码(16进制)            |
| %    | 字符 % 或类型转换字符前缀 |

具体例子：

```Java
/* Formatter 对各种数据类型转换的荔枝 */  
public class Conversion {  
    public static void main(String[] args) {  
        Formatter f = new Formatter(System.out);  
  
        char u = 'a';    
        System.out.println("u = 'a'"); // u = 'a'  
          
        f.format("%%s: %s\n", u); // %s: a  
          
        f.format("%%c: %c\n", u); // %c: a  
        f.format("%%b: %b\n", u); // %b: true  
        f.format("%%h: %h\n", u); // %h: 61  
//       f.format("d: %d\n", u); //  java.util.IllegalFormatConversionException: d != java.lang.Character  
//       f.format("f: %f\n", u); // java.util.IllegalFormatConversionException: f != java.lang.Character  
//       f.format("e: %e\n", u); // java.util.IllegalFormatConversionException: e != java.lang.Character  
//       f.format("x: %x\n", u); // java.util.IllegalFormatConversionException: x != java.lang.Character  
  
        int v = 121;  
        System.out.println();  
        System.out.println("v = 121"); // v = 121  
          
        f.format("%%d: %d\n", v); // %d: 121  
        f.format("%%c: %c\n", v); // %c: y  
        f.format("%%b: %b\n", v); // %b: true   
        f.format("%%s: %s\n", v); // %s: 121  
        f.format("%%x: %x\n", v); // %x: 79  
        f.format("%%h: %h\n", v); // %h: 79   
//       f.format("f: %f\n", v); // java.util.IllegalFormatConversionException: f != java.lang.Integer  
//       f.format("e: %e\n", v); // java.util.IllegalFormatConversionException: e != java.lang.Integer  
  
        BigInteger w = new BigInteger("50000000000000");  
        System.out.println();  
        System.out.println("w = new BigInteger(\"50000000000000\")"); // w = new BigInteger("50000000000000")  
        f.format("%%d: %d\n", w); // %d: 50000000000000  
        f.format("%%b: %b\n", w); // %b: true  
        f.format("%%s: %s\n", w); // %s: 50000000000000  
        f.format("%%x: %x\n", w); // %x: 2d79883d2000  
        f.format("%%h: %h\n", w); // %h: 8842a1a7  
//       f.format("c: %c\n", w); // java.util.IllegalFormatConversionException: c != java.math.BigInteger  
//       f.format("f: %f\n", w); // java.util.IllegalFormatConversionException: f != java.math.BigInteger  
//       f.format("e: %e\n", w); // java.util.IllegalFormatConversionException: e != java.math.BigInteger  
  
        double x = 179.543;  
        System.out.println();  
        System.out.println("x = 179.543"); // x = 179.543  
        f.format("%%b: %b\n", x); // %b: true  
        f.format("%%s: %s\n", x); // %s: 179.543  
        f.format("%%f: %f\n", x); // %f: 179.543000  
        f.format("%%e: %e\n", x); //%e: 1.795430e+02, 科学表示法  
        f.format("%%h: %h\n", x); // %h: 1ef462c  
//       f.format("d: %d\n", x); // java.util.IllegalFormatConversionException: d != java.lang.Double  
//       f.format("c: %c\n", x); // java.util.IllegalFormatConversionException: c != java.lang.Double  
//       f.format("x: %x\n", x); // java.util.IllegalFormatConversionException: x != java.lang.Double  
  
        Conversion y = new Conversion();  
        System.out.println();  
        System.out.println("y = new Conversion()"); // y = new Conversion()  
        f.format("%%b: %b\n", y); // %b: true  
        f.format("%%s: %s\n", y); // %s: chapter13.Conversion@4aa298b7  
        f.format("%%h: %h\n", y); // %h: 4aa298b7  
//       f.format("d: %d\n", y); // java.util.IllegalFormatConversionException: d != chapter13.Conversion  
//       f.format("c: %c\n", y); // java.util.IllegalFormatConversionException: c != chapter13.Conversion  
//       f.format("f: %f\n", y); // java.util.IllegalFormatConversionException: f != chapter13.Conversion  
//       f.format("e: %e\n", y); // java.util.IllegalFormatConversionException: e != chapter13.Conversion  
//       f.format("x: %x\n", y); // java.util.IllegalFormatConversionException: x != chapter13.Conversion  
  
        boolean z = false;  
        System.out.println();  
        System.out.println("z = false"); // z = false  
        f.format("%%b: %b\n", z); // %b: false  
        f.format("%%s: %s\n", z); // %s: false  
        f.format("%%h: %h\n", z); // %h: 4d5  
//       f.format("d: %d\n", z); // java.util.IllegalFormatConversionException: d != java.lang.Boolean  
//       f.format("c: %c\n", z); // java.util.IllegalFormatConversionException: c != java.lang.Boolean  
//       f.format("f: %f\n", z); // java.util.IllegalFormatConversionException: f != java.lang.Boolean  
//       f.format("e: %e\n", z); // java.util.IllegalFormatConversionException: e != java.lang.Boolean  
//       f.format("x: %x\n", z); // java.util.IllegalFormatConversionException: x != java.lang.Boolean  
    }  
} 
```

## 正则表达式

要想学好正则表达式，基本上要记住的就是一堆语法。

废话不多说，直接看如何创建正则表达式。

**字符**

| 正则表达式 | 说明                              |
| ---------- | --------------------------------- |
| B          | 指定字符B                         |
| \xhh       | 十六进制值为0xhh的字符            |
| \uhhhh     | 十六进制表示为0xhhhh的Unicode字符 |
| \t         | 制表符Tab                         |
| \n         | 换行符                            |
| \r         | 回车                              |
| \f         | 换页                              |
| \e         | 转义                              |

**字符类**

| 正则表达式   | 说明                            |
| ------------ | ------------------------------- |
| .            | 任意字符                        |
| [abc]        | 包含a、b和c的任何字符           |
| [^abc]       | 除了a、b和c之外的任何字符(否定) |
| [a-zA-Z]     | 从a到z或从A到Z的任何字符(范围)  |
| [abc[hij]]   | 任意abchij字符                  |
| [a-z&&[hij]] | 任意h、i或j(相交)               |
| \s           | 空白符(空格、tab、换行和回车)   |
| \S           | 非空白符                        |
| \d           | 数字[0-9]                       |
| \D           | 非数字`[^0-9]`                  |
| \w           | 词字符([a-zA-Z0-9])             |
| \W           | 非词字符                        |

**逻辑操作符**

| 正则表达式 | 说明      |
| ---------- | --------- |
| XY         | Y跟在后面 |
| `X\|Y`     | X或Y      |
| (X)        | 捕获组    |

**量词**

| 贪婪型 | 勉强型  | 占有型  |       如何匹配        |
| :----: | :-----: | :-----: | :-------------------: |
|   X?   |   X??   |   X?+   |      一个或零个X      |
|   X*   |   X*?   |   X*+   |      零个或多个X      |
|   X+   |   X+?   |   X++   |      一个或多个X      |
|  X{n}  |  x{n}?  |  X{n}+  |       恰好n次X        |
| X{n,}  | X{n,}?  | X{n,}+  |       至少n次X        |
| X{n,m} | X{n,m}? | X{n,m}+ | X至少n次，且不超过m次 |

表达式 X 通常必须用圆括号括起来。

#### Pattern 和 Matcher

1. Pattern.compile(regex) 编译 `regex` 并产生 `Pattern` 对象。
2. Pattern.matcher(检索的字符串)生成一个 `Matcher` 对象。

**基本用法**

```Java
public class TestRegularExpression {  
    public static void main(String[] args) {  
        String[] array = {"aabbcc", "aab", "aab+", "(b+)"};  
          
        for (String arg : array) {  
            System.out.println();  
            print("Regular expression: \"" + arg + "\"");  
            Pattern p = Pattern.compile(arg); // step1: Pattern 表示编译后的匹配模型Pattern.（编译后的正则表达式）  
            Matcher m = p.matcher("aabbcc"); // step2: 模型实例 检索 待匹配字符串并 生成一个匹配对象Matcher， Matcher有很多方法  
            while (m.find()) {  
                print("Match \"" + m.group() // 待匹配的字符串  
                                 + "\" at positions "   
                                 + m.start() // 字符串匹配regex的起始位置  
                                 + "-" + (m.end() - 1)); // 字符串匹配regex的终点位置  
            }  
        }  
    }  
} 
```

**Matcher 方法**

```Java
boolean matches(); //判断 输入字符串 是否匹配正则表达式regex；  
boolean lookingAt(); //判断输入字符串（不是整个）的开始部分是否匹配 regex；  
boolean find(); //用于 在 CharSequence 输入字符串中查找多个匹配；  
boolean find(int start);  //用于在 CharSequence 输入字符串的start 位置开始查找多个匹配；  
String group(); //用于返回匹配regex的输入字符串的子串；
```

**常用的 Pattern 标记**

```java
public static Pattern compile(String regex, int flags) {  
    return new Pattern(regex, flags);  
} 
// Pattern.CASE_INSENSITIVE：  不区分大小写；
// Pattern.MULTILINE： 允许多行，即不以换行字符作为分隔符；
// Pattern.COMMENTS： 模式中允许空格和注释， 不以空格和注释作为分隔符
```

**Pattern.split()**

将字符串分割成符合 `regex` 的字符数组。

```Java
public class SplitDemo {  
    public static void main(String[] args) {  
        String input = "This!!unusual use!!of exclamation!!points";  
        print(Arrays.toString(Pattern.compile("!!").split(input))); // split(input, 0); 对匹配次数不做任何限制  
  
        /* (只匹配前2个 !! ) */  
        /* 注意：分割边界在分割结果中被删除 */  
        print(Arrays.toString(Pattern.compile("!!").split(input, 3))); // 限定匹配次数，limit限制将输入字符串分割成数组的数组大小  
    }  
}    
/* 
[This, unusual use, of exclamation, points] 
[This, unusual use, of exclamation!!points] 
*/  
```

## 扫描输入

使用 `BufferedReader` 实现。

```Java
public class SimpleRead {  
    public static BufferedReader input = new BufferedReader(new StringReader(  
            "Sir Robin of Camelot\n22 1.61803"));  
  
    public static void main(String[] args) {  
        try {  
            System.out.println("\n1.What is your name?");  
            String name = input.readLine();  
            System.out.println(name); // Sir Robin of Camelot  
              
            System.out.println("\n2.input: <age> <double>");  
            String numbers = input.readLine();  
            System.out.println("input.readLine() = " + numbers); // 22 1.61803  
              
            String[] numArray = numbers.split(" ");  
            int age = Integer.parseInt(numArray[0]); // 22  
            double favorite = Double.parseDouble(numArray[1]); // 1.61803  
              
            System.out.format("Hi %s.\n", name);  
            System.out.format("In 5 years you will be %d.\n", age + 5);  
            System.out.format("My favorite double is %f.", favorite / 2);  
        } catch (IOException e) {  
            System.err.println("I/O exception");  
        }  
    }  
}  
// output:
/* 
 1.What is your name? 
Sir Robin of Camelot 
 
2.input: <age> <double> 
input.readLine() = 22 1.61803 
Hi Sir Robin of Camelot. 
In 5 years you will be 27. 
My favorite double is 0.809015. 
 */ 
```

使用 `Scanner` 实现

```Java
public class BetterRead {  
    public static void main(String[] args) {  
        // Scanner 可以接受任何类型的 Readable 输入对象  
        Scanner stdin = new Scanner(SimpleRead.input);  
        System.out.println("What is your name?");  
        // 所有的输入，分词以及翻译的操作都隐藏在不同类型的 next 方法 中.  
        String name = stdin.nextLine(); // nextLine() 返回 String  
        System.out.println(name);  
          
        System.out.println("How old are you? What is your favorite double?");  
        System.out.println("(input: <age> <double>)");  
          
        // Scanner 直接读入 integer 和 double 类型数据  
        int age = stdin.nextInt();  
        double favorite = stdin.nextDouble();  
        System.out.println(age);  
        System.out.println(favorite);  
          
        System.out.format("Hi %s.\n", name);  
        System.out.format("In 5 years you will be %d.\n", age + 5);  
        System.out.format("My favorite double is %f.", favorite / 2);  
    }  
} 
// output:
/* 
What is your name? 
Sir Robin of Camelot 
How old are you? What is your favorite double? 
(input: <age> <double>) 
22 
1.61803 
Hi Sir Robin of Camelot. 
In 5 years you will be 27. 
My favorite double is 0.809015. 
```

## 总结

过去， `Java` 对字符串操作的支持相当不完善。不过随着近几个版本的升级，我们可以看到， `Java` 已经从其他语言中吸取了许多成熟的经验。到目前为止，它对字符串操作的支持已经很完善了。不过，有时你还要在细节上注意效率的问题，例如恰当地使用 `StringBuilder` 等。

## 感谢

[thinking-in-java(13) String字符串](https://blog.csdn.net/pacosonswjtu/article/details/78631796)