title: Thinking-in-Java-读书笔记-3-操作符
date: 1994.02.26 20:46:25
categories:
- Thinking In Java 读书笔记
tags:
- Thinking In Java
- 读书笔记
---

**在最底层，Java 中的数据是通过使用操作符来操作的。**

 作为一个开发将近一年的程序员，对这些基本的操作符的掌握还是算熟练的，因此就不过多介绍了，毕竟大家基本上都知道，但是还是把内容过了一遍，笔记就不赘述了。一切的理论都不如实践来的实际，遇到模棱两可的，不如直接通过程序跑一下进行验证。
 
<!-- more -->

## 1. 赋值

使用「=」，意思是 **取右边的值，把它复制给左边，右值可以是任何数、变量或者是表达式，但左值必须是一个明确的已命名的变量。**

        Student a = new Student();
        Student b = new Student();
        a.name = "a";
        b.name = "b";
        a = b;
        System.out.println("a->" + a + "\tb->" + b);
        b.name = "c";
        System.out.println("a->" + a + "\tb->" + b);
        System.out.println("---------------");

输出：

    a->b	b->b
    a->c	b->c

对象的赋值一般是值引用的传递。

## 2. 使用操作符时常犯的错

使用操作符时一个常犯的错就是，即使对表达式如何计算有点不确定，也不愿意使用括号。

    while (x = y) {
        // ...
    }
**x = y** 属于合法表达式，但是结果是 int 类型的值，二 while 括号需要的是 boolean 值，因此在编译期即会报错。

## 3. 截尾和舍入

    int a = 0.6;
    int b = 0.4;
    int c = -0.6;
    int d = -0.4;

最后的结果都是 0；

## 4. 运算溢出

        int big = Integer.MAX_VALUE;
        System.out.println("big = " + big);
        int bigger = big * 2;
        System.out.println("bigger = " + bigger);

输出结果

    big = 2147483647
    bigger = -2

虽然溢出了，但是并没有报错，运行时也不会出现异常。

## 总结

对基本运算符的掌握是程序必备的技能，无论哪门语言基本上相通的，越基础的东西，越是需要搞透彻，在开发的过程中，可不能因为这么基础的东西犯错！
