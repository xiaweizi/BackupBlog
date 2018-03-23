---
title: 『Thinking in Java 读书笔记』—— 12-通过异常处理错误
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_12
date: 1994-03-11 15:13:33

---

[Thinking in java 读书笔记](http://xiaweizi.cn/categories/Thinking-In-Java-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)

> `Java` 的基本理念是 「结构不佳的代码不能运行。

发现错误的理想时机是在编译阶段，也就是在你试图运行程序之前。然而，编译期间并不能找出所有的错误，余下的问题必须在运行期间解决。这就需要错误源通过某种方式，把适当的信息传递给某个接受者—接收者将知道如何正确处理这个问题。

<!-- more -->



## 基本异常

**异常情形**是指阻止当前方法或作用域继续执行的问题。除法就是一个简单的例子。除数有可能为 0，所以先进行检查很有必要。但除数为 0 代表的究竟是什么意思？通过当前正在解决的问题环境，或许能知道该如何处理除数为 0 的情况。但如果这是一个意料之外的值，你也不清楚该如何处理，那就抛出异常，而不是顺着原来的路径继续执行下去。

```java
class ExceptionTest {
    public static void main(String[] args) {
        int a = 0;
        try {
            System.out.println("1111");
            if (a == 0) {
                System.out.println("2222");
                throw new NullPointerException("a cannot be 0");
            }
        } catch (Exception e) {
            System.out.println("3333");
            e.printStackTrace();
            System.out.println("e:" + e.getMessage());
        } finally {
            System.out.println("4444");
        }
    }
}
```

上面就是简单的异常处理情况，对于 `a==0` 这种 **异常情况**，我们主动抛出异常,异常的抛出方式类似 `Java`创建对象的过程，直接 `new NullPointerException()`的方式创建我们非常熟悉的 **空指针** 异常。

这个时候编译器会提示你需要将抛出的异常 `try catch` 掉。那么就按照提示对出现异常的位置进行捕获。在 `catch` 的地方进行异常的处理。

`finally` 块内的代表着无论是否出现异常，这段代码块的内容都会执行。

看一下运行效果：

```Java
java.lang.NullPointerException: a cannot be 0
1111
2222
3333
e:a cannot be 0
4444
	at chapter12.ExceptionTest.main(ExceptionTest.java:20)
```

## 创建自定义异常

不必拘泥于 `Java` 中已有的异常类型。 `Java` 提供的异常体系不可能预见所有的希望加以报告的错误，所以可以自己定义异常类来表示程序中可能会遇到的特定问题。

```Java
class CustomException {

    static void f() throws MyException {
        System.out.println("1111");
        throw new MyException();
    }

    public static void main(String[] args) {
        try {
            System.out.println("2222");
            f();
        } catch (MyException e) {
            System.out.println("3333");
            e.printStackTrace();
        }
    }
}

class MyException extends Exception {

}
```

## 异常的栈轨迹

`printStackTrace()` 方法所提供的信息可以通过 `getStackTrace()` 方法来直接访问，这个方法将返回一个由栈轨迹中的元素所构成的数组，其中每一个元素都表示栈中的一帧。看一下代码演示：

```Java
class StackTraceTest {
    static void f() {
        try {
            throw new NullPointerException("null");
        } catch (Exception e) {
            for (StackTraceElement stackTraceElement : e.getStackTrace()) {
                System.out.println(stackTraceElement.getMethodName());
            }

        }
    }

    static void g() {
        f();
    }

    static void h() {
        g();
    }

    public static void main(String[] args) {
        f();
        System.out.println("----------");
        g();
        System.out.println("----------");
        h();
    }
}
```

输出结果：

```
f
main
----------
f
g
main
----------
f
g
h
main
```

一层一层的展示异常栈内的信息。

## finally

对于一些代码，可能会希望无论 `try` 块中的异常是否抛出，它们都能得到执行。这通常适用于内存回收之外的情况。为了达到这个效果，可以在异常处理程序后面加上 `finally` 子句。

```java
class FinallyTest {

    static void f(int i) {
        try {
            System.out.println("111");
            if (i == 1) {
                return;
            }
            System.out.println("222");
            if (i ==2) {
                return;
            }
            System.out.println("222");
            if (i ==3) {
                return;
            }
            System.out.println("end");

        } finally {
            System.out.println("finally clean up");
        }
    }
    public static void main(String[] args) throws Exception{
        for (int i = 1; i < 5; i++) {
            f(i);
        }
        System.out.println("5555555");
        throw new NullPointerException("dd");
    }
}
```

输出结果：

```Java
Exception in thread "main" java.lang.NullPointerException: dd
	at chapter12.FinallyTest.main(FinallyTest.java:40)
111
finally clean up
111
222
finally clean up
111
222
222
finally clean up
111
222
222
end
finally clean up
5555555
```

## 总结

异常是 `Java` 程序设计不可分割的一部分，如果不了解如何使用它们，那你只能完成很有限的工作。异常处理的有点之一就是它使得你可以在某处集中精力处理你要解决的问题，而在另外一处处理你编写的这段代码中产生的错误。尽管异常通常被认为是一种工具，使得你可以在运行时报告错误并从错误中恢复。

