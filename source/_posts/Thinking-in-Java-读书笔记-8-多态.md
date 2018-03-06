title: Thinking-in-Java 读书笔记-8-多态
author: 下位子
date: 1994-03-07 15:13:33
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
---

## 前言

记得刚接触`Java`的时候，整天被老师灌输的就是「封装」、「继承」、「抽象」和「多态」，因为这是面向对象语言基本的特征。尴尬的是，直到大学毕业了也没有彻底搞懂...

多态通过分离「做什么」和「怎么做」，从另一个角度将接口和实现分离开来。多态不但能够改善代码的组织结构和可读性，还能创建可扩展的程序--即无论在项目最初创建时还是需要添加新功能都可以生长的程序。

「封装」通过合并特征和行为来创建新的数据类型，实现隐藏则通过将细节私有化把接口和实现分离开来，这种类型的组织机制对那些拥有过程化程序设计背景的人来说，更容易理解。而「多态」的作用则是消除类型之间的耦合关系。

继承允许将对象视为它自己本身的类型或其他类型来加以处理。这种能力即为重要，因为它允许将多种类型(从同一基类导出的)视为同一类型处理。多态方法调用允许一种类型表现出与其他相似类型之间的区别，只要它们都是从同一基类导出而来的。

## 定位具体子类

```
class Machine {
    void work() {
        System.out.println("Machine work!");
    }
}

class Computer extends Machine {
    @Override
    void work() {
        System.out.println("Computer work");
    }
}

class Light extends Machine {
    @Override
    void work() {
        System.out.println("Light work");
    }

    public static void main(String[] args) {
        run(new Computer());
        run(new Light());
    }

    static void run(Machine machine) {
        machine.work();
    }
}
```

`run`方法传入的是基类类型的`Machine`，这样就不需要创建适配各种类型的`run`方法，加强了可维护性和代码健壮性。利用了「向上转型」这一特性，不用在乎具体的实现，只需要传入接口即可。

`machine.work();`如编译器是怎么知道这个`machine`的引用是指向具体的实现的呢？「后期绑定」，它的含义就是运行时根据对象的类型进行绑定，后期绑定也叫做动态绑定或运行时绑定。编译器一直不知道对象的类型，但是方法调用机制能找到正确的方法体，并加以调用。

Java 中除了`static`和`final`方法之外，其他所有的方法都是后期绑定，这意味着通常情况下，我们不必判定是否应该进行后期绑定。

## 缺陷

**不能覆盖私有方法**

**域与静态方法**

## 构造器的调用顺序

```
class Animal {
    Animal() {
        System.out.println("Animal");
    }
}

class Car {
    Car() {
        System.out.println("Car");
    }
}

class Water {
    Water() {
        System.out.println("Water");
    }
}

class People extends Animal{
    People() {
        System.out.println("People");
    }
}

class Student extends People {
    Student() {
        System.out.println("Student");
    }

    private Car car = new Car();
    private Water water = new Water();

    public static void main(String[] args) {
        People student = new Student();
    }
}
```
运行结果：

```
Animal
People
Car
Water
Student
```


顺序：

1. 调用基类的构造器
2. 按声明顺序调用成员的初始化方法
3. 调用导出类构造器的主体

## 构造器内部的多态方法的行为

```
class Phone {

    Phone() {
        System.out.println("before call");
        call();
        System.out.println("after call");
    }

    void call() {
        System.out.println("Phone Call");
    }
}

class IPhone extends Phone {
    private String num = "110";
    IPhone(String num) {
        this.num = num;
        System.out.println("number: " + this.num);
    }

    @Override
    void call() {
        System.out.println("call number: " + num);
    }

    public static void main(String[] args) {
        IPhone iPhone = new IPhone("119");
    }
}

```
运行结果：

```
before call
call number: null
after call
number: 119
```
从结果可以看出，在执行基类的构造函数调用到多态方法时，会调用被覆盖的具体方法，但这个时候子类并没有完成初始化，导致获取到子类的成员是`null`。

初始化的实际过程：

1. 在其他任何事物发生之前，将分配个对象的存储空间初始化成二进制的零。
2. 如前所述那样调用基类构造器
3. 按照声明的顺序调用成员的初始化方法
4. 调用导出类的构造器主体

## 向下转型

由于向上转型会丢失具体的类型信息，所以我们就像，通过向下转型应该能获取到类型数据。然而，我们知道向上转型是安全的，因为基类不会具有大余导出类的接口。因此，我们通过基类接口发送的消息保证都可以接受。但是对于向下转型，就无法知道是哪个具体的对象了。

因此需要进行强制转换，但是如果强转失败会抛出`ClassCaseException`的异常，因此在使用向下转型的时候需要进行类别的判断，进行容错处理。

## 总结

多态意味着不同的形式。我们持有从基类继承而来的相同接口，以及使用该接口的不同形式：不同版本的动态绑定方法。

为了在自己的程序中有效地运用多态乃至面向对象的技术，必须扩展自己的编程视野，使其不仅包括个别类的成员和消息，而且要包括类与类之间的共同特性以及它们之间的关系。尽管这需要极大的努力，但是这样做是非创值得的，因为它可以带来很多成效：更快的程序开发过程、更好的代码组织、更好的扩展的程序以及更容易的代码维护等。

