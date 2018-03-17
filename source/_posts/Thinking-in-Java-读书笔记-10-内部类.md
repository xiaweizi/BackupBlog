---
title: Thinking-in-Java 读书笔记-10-内部类
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_10
date: 1994-03-09 15:13:33

---

[Thinking in java 读书笔记](http://xiaweizi.cn/categories/Thinking-In-Java-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)

> 可以将一个类的定义放在另一个类的定义内部，这就是内部类。

## 前言

内部类非常的有用，因为它允许你把一些逻辑相关的类组织在一块，并控制位于内部的类的可视性，内部类和组合是完全不同的概念，这点很重要。

<!-- more -->

```
class Outer {
    int a =2;

    int f () {
        System.out.println("Outer.f");
        return a;
    }

     class Inner {
        Outer outer() {
            System.out.println("outer a = " + a);
            return Outer.this;
        }
    }

     private Inner inner() {
        return new Inner();
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.inner().outer().f();
    }
}
```

创建起来也是很方便，非`static`内部类是可以持有外部类的引用，也就是使用外部类的成员变量和成员方法。

可以使用`Outer.this`拿到外部类的对象

## 方法内部嵌套内部类

```
class People {

    interface IWork {
        void work();
    }

    IWork teach(final String content) {

        class Student implements IWork{

            @Override
            public void work() {
                System.out.println("content:" + content);
            }
        }
        return new Student();
    }

    public static void main(String[] args) {
        People people = new People();
        people.teach("English").work();
    }
}
```

方法 内部也是可以嵌套内部类的，还可以使用方法的参数，不过参数要加上`final`修饰，表示不可更改对象值。

## 匿名内部类

顾名思义，就是没有名字的内部类，这种类型的内部类主要应用在不需要创建具体的实现类来实现接口，直接通过`new`的方式，在内部类实现。

```
abstract class Base {
    Base(int i) {
        System.out.println("construct: " + i);
    }
    abstract void f();
}

public class AnonyConstruct {
    public static Base getBase() {
        return new Base(2) {
            {
                System.out.println("dddd");
            }
            @Override
            void f() {
                System.out.println("Anony f");
            }
        };
    }

    public static void main(String[] args) {
        Base base = getBase();
        base.f();
    }
}
```

`Base`作为借口没有具体的实现内容，但是可以通过匿名内部类的方式实现具体的接口，又利用向上转型这一特性完成对接口的引用。这种匿名内部类可以有构造函数的，不过只能是这种无参的构造函数，

## 在看工厂模式

```
interface IService {
    void method1();
    void method2();
}
interface IFactory {
    IService getService();
}

class Service1 implements IService {

    @Override
    public void method1() {
        System.out.println("Service1 method 1");
    }

    @Override
    public void method2() {
        System.out.println("Service1 method 2");
    }

    static IFactory factory = new IFactory() {
        @Override
        public IService getService() {
            return new Service1();
        }
    };
}

class Service2 implements IService {
    @Override
    public void method1() {
        System.out.println("Service2 method 1");
    }

    @Override
    public void method2() {
        System.out.println("Service2 method 2");
    }

    static IFactory factory = new IFactory() {
        @Override
        public IService getService() {
            return new Service2();
        }
    };
}

class Main {

    static void factoryConsumer(IFactory factory) {
        IService service = factory.getService();
        service.method1();
        service.method2();
    }

    public static void main(String[] args) {
        factoryConsumer(Service1.factory);
        factoryConsumer(Service2.factory);
    }
}
```


直接上代码，第一步创建产品和工厂的实现接口，这步是不变的，变化的是产品的具体实现中，构造了匿名内部类完成工厂具体的实现。通过这种方式也可以完成工厂模式。

## 嵌套类

如果不需要内部类对象与其外围类对象之间有关系，那么可以将内部了声明成`static`，这中通常被称为「嵌套类」，想要理解`static`应用内部类的含义，就必须记住，普通的内部类对象隐士的保存了一个引用，指向创建它的外围类对象，然而，当内部类是`static`时，就不是这样的了，嵌套类意味着：

1. 要创建嵌套类的对象，并不需要其外围类的对象
2. 不要从嵌套类的对象中访问非静态的外围类对象

```
class StaticOuter {
    int a = 1;

    StaticInner getInner() {
        return new StaticInner();
    }

    static class StaticInner {
        int b = 12;
        void f() {
            System.out.println("static inner f");
            System.out.println("b " + b);
        }
    }

    public static void main(String[] args) {
        new StaticOuter().getInner().f();
    }
}

```

在`Android`中的应用就是使用`Handler`的时候，如果直接`new Handler`的方式创建`handler`，是很容易造成空指针异常，那这个时候就需要创建内部类，但是又容易造成内存泄漏，因此就采用`static`的方式，这样没有拿到外部类的引用，就避免了内存泄漏的发生。

## 为什么需要内部类

致辞，我们已经看到许多描述内部类的语法和语义，但是这并不能回答 为什么需要内部类。

一般来说，内部类继承自某个类和实现某个接口，内部类的代码操作创建它的外围类的对象，所以可以认为内部类提供了某种进入其外围类的窗口。

**每个内部类都能独立地继承自一个实现，所以无论外围类是否已经继承了某个实现，对于内部类都没有影响。**

如果没有内部类的提供，可以继承多个具体的抽象能力，一些设计与编程问题就很难解决。从这个角度，内部类使得多重继承的解决方案变得完整。接口解决了部分问题，而内部类有效的实现了多重继承。也就是说，内部类允许继承多个非接口类型。

## 总结

为什么我们仍然使用局部内部类而不是匿名内部类，唯一的理由就是，我们需要一个已命名的构造器，或者需要重载构造器，而匿名内部类只能用于实例化。，所以使用局部内部类而不是用匿名内部类的另一个理由就是，需要不止一个该内部类的对象。