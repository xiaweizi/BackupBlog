title: Thinking-in-Java 读书笔记-7-复用类
author: 下位子
date: 1994-03-06 15:13:33
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
---

> 复用代码是 Java 众多引人注目的功能之一。但要想成为极具革命性的语言，仅仅能够复制diamante并对之加以改变是不够的，它还必须能够做更多的事情。

## 组合用法

这个用户最为常用，即在新建的类中，持有别的对象的引用。假设你需要某个对象，它要具有多个 String 对象，几个基本类型数据，以及另一个了类的对象，这种使用在开发中最为常见。对于非基本类型的对象，必须将其引用置于新的类中。

<!-- more -->

## 继承用法

组合的用法比较平实，但是继承使用的是一种特殊的语法。在继承过程中，需要先声明新类与旧类相似。

```
class Main {

    public static void main(String[] args) {
        Student student = new Student();
    }
}

class People {
    People() {
        System.out.println("People");
    }
    People(String name) {
        System.out.println("People" + name);
    }
}

class Worker extends People{
    Worker() {
        System.out.println("Worker");
    }
    Worker(String name) {
        super(name);
        System.out.println("Worker" + name);
    }
}

class Student extends Worker {
    Student() {
        System.out.println("Student");
    }

    Student(String name) {
        super(name);
        System.out.println("Student" + name);
    }
}
```

运行结果：

```
People
Worker
Student
```

这种继承关系的又叫做父子关系，子类继承了所有父类的特点，即公开成员变量和公开的方法，当调初始化子类构造函数的时候，会默认调用父类的无参构造函数。当然你也可以通过`super`的方式主动选择调用父类的某个构造函数。

## 组合加继承的方式

同时使用组合和继承也是非常常见的事。

```
class Shape {

    public static void main(String[] args) {
        CADSystem system = new CADSystem(4);
        try {
            // ..
        } finally {
            system.dispose();
        }
    }

    private int age;

    Shape(int i) {
        System.out.println("Shape construct");
        age = i;
    }

    void dispose() {
        System.out.println("Shape dispose");
    }

    @Override
    public String toString() {
        return "age:" + age;
    }
}

class CADSystem extends Shape {

    private line[] lines = new line[3];
    private Circle circle;
    private Rect rect;
    CADSystem(int i) {
        super(i);
        for (int j = 0; j < lines.length; j++) {
            lines[j] = new line(j);
        }
        circle = new Circle(i);
        rect = new Rect(i);
        System.out.println("CADSystem construct");
    }

    @Override
    void dispose() {
        for (line line : lines) {
            line.dispose();
        }
        circle.dispose();
        rect.dispose();
        super.dispose();
    }
}

class line extends Shape {

    line(int i) {
        super(i);
        System.out.println("Line construct " + i);
    }

    @Override
    void dispose() {
        System.out.println("line dispose");
        super.dispose();
    }
}

class Circle extends Shape {

    Circle(int i) {
        super(i);
        System.out.println("Circle construct");
    }

    @Override
    void dispose() {
        System.out.println("Circle dispose");
        super.dispose();
    }
}

class Rect extends Shape {

    Rect(int i) {
        super(i);
        System.out.println("Rect construct");
    }

    @Override
    void dispose() {
        System.out.println("Rect dispose");
        super.dispose();
    }
}
```

其实有点像类的适配器模式，模拟这样的一种行为，平时画画结束的时候，需要对资源进行清理。`Line` `Circle` 和 `Rect`都是画图工具，同时继承了`Shape`这个基类，并继承了`dispose`释放方法。

`CADSystem` 也继承了`Shape`，并同时用三种工具的引用。在创建`CADSystem`的时候也完成了工具的初始化，释放资源的时候，遍历所有工具并释放资源。

## 在组合和继承之间选择

组合和继承都允许在新的类中放置子对象，组合是显示地这样做，而继承是隐式的做。

组合技术通常用于想在新的类中使用现有类的功能而非它的接口这种情况。即在新的类中嵌入某个对象，让其实现所需要的功能，但新的类用户看到的只是为新类所定义的接口，而非所嵌入对象的接口。

继承侧重的是新类和基类之间的关系，这种关系可以用新类是现有类的一种类型这句话加以概括。
这个时候要提到一个用语「向上转型」，在继承图中可以看出，基类位于上端，子类位于下端，子类转成基类就是「向上转型」，因此向上转型是安全的。到这也可以这么说，一个最清晰的判断是用继承还是组合的办法：问一问自己是否需要从新类向基类进行向上转型，如果必须向上转型，则继承是必须的，但如果不需要，则应当好好考虑是否需要继承。

## final

指无法改变的，不想做改变可能出于两种理由：设计或效率。`final`修饰基本类型时，数值恒定不变；修饰对象引用，引用恒定不变。修饰方法，子类不能重写该方法，修饰类，该类不能被继承。


## 总结

继承和组合都能从现有类型生成新类型，组合一般是将现有类型作为新类型底层实现的一部分加以复用，二继承复用的是接口。

在使用继承时，由于导出类具有基类接口，因此它可以向上转型，这对多态来讲至关重要。

尽管面向对象编程对继承极力强调，但在开始一个设计时，一般优先选择使用组合，只在确实必要时才使用继承。因为组合更具有灵活性，此外，通过对成员类型使用继承技术的添加技巧，可以在运行时改变那些成员对象的类型和行为。

当你开始设计一个系统时，应该认识到程序开发是一个增量过程，犹如人类的学习一样，这一点很重要。
