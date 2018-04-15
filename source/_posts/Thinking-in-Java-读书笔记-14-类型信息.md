---
title: 『Thinking in Java 读书笔记』—— 14-类型信息
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_14
date: 1994-04-15 16:13:33

---

[Thinking in java 读书笔记](http://xiaweizi.cn/categories/Thinking-In-Java-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)

> 运行时类型信息使得你可以在程序运行时发现和使用类型信息。
>
> Java 通过两种方式在运行时识别对象和类信息的，一种是 **传统的RTTI**，(RunTime Type Identification 运行时类型定义)，假定我们在编译时已经知道了所有的类型。第二种是 **反射机制**，它允许在运行时发现和使用类信息。

## 为什么需要 RTTI

**啥叫多态**，父类方法在子类可能会被覆盖，但是由于方法是动态绑定的，所以即使是通过泛化的父类引用来调用方法，也是可以产生正确的行为，这就是多态。

<!-- more -->

```Java
// 父类  
abstract class Shape {  
    void draw() {  
        System.out.println(this + ".draw()");  
    }  
  
  
    abstract public String toString();  
}  
// 子类  
class Circle extends Shape {  
    public String toString() {  
        return "Circle";  
    }  
}  
//子类  
class Square extends Shape {  
    public String toString() {  
        return "Square";  
    }  
}  
//子类  
class Triangle extends Shape {  
    public String toString() {  
        return "Triangle";  
    }  
}  
// 利用父类引用调用子类方法的荔枝  
public class Shapes {  
    public static void main(String[] args) {  
        // 把 Shape子类数组转换为 泛型为Shape的List容器  
        List<Shape> shapeList = Arrays.asList(new Circle(), new Square(), new Triangle());  
        for (Shape shape : shapeList)  
            shape.draw();  
          
        System.out.println();  
        for (Shape shape : shapeList)             
            System.out.println("does " + shape.getClass().getName() + " belong to Circle = " + rotate(shape));  
    }  
      
    static boolean rotate(Shape s) {  
        if(s instanceof Circle) {  
            return true;  
        } else {  
            return false;  
        }  
    }  
}   
/* 
Circle.draw() 
Square.draw() 
Triangle.draw() 
 
 
does chapter14.Circle belong to Circle = true 
does chapter14.Square belong to Circle = false 
does chapter14.Triangle belong to Circle = false 
*/
```

**RTTI的基本形式**，当从数组中取出元素时， `RTTI` 会将 `Object` 转换成 `Shape` 类型， `RTTI` 类型转换并不彻底，因为把所有的 `Object` 转换成 `Shape` 父类，而不是转换到 `Circle` 或 `Square` 或 `Triangle` 。

## Class 对象

- 类型信息在运行时是如何表示的：这项工作是由 Class 对象的特殊对象完成的。
- Class 对象的作用：用来创建类的所有常规对象和保存对象在运行时的类型信息。
- Java 使用 Class 对象来实现 RTTI，即类型转换。
- 每个类都有一个 Class 对象：每当编写并且编译一个新类，就会产生一个 Class 对象。
- 所有的类都是在第一次使用时，动态加载到 jvm 中的：当程序创建第一个对类的静态成员的引用时，就会加载这个类。
- Java 程序在它运行之前并非被完全加载，其各个部分是在必需时才加载。
- 类加载器加载 Class 对象：类加载器首先检查这个类的 Class 对象是否已经被加载。如果没有加载，则默认的类加载器会根据类名查找 `.class` 文件。
- 一旦某个类的 Class 对象被载入内存，他就被用来创建这个类的所有对象。

```Java
class Name {
    static {
        System.out.println("name is loading");
    }
}
class Sex {
    static {
        System.out.println("sex is loading");
    }
}
class Age {
    public static String desc = "desc";
    static {
        System.out.println("age is loading");
    }
    Age() {
        System.out.println("age construct");
    }
}

public class ClassTest {
    public static void main(String[] args) {
        new Name();
        try {
            if (Boolean.TYPE == boolean.class) {
                System.out.println(true);
            }
            Class.forName("chapter14.Sex");
        } catch (Exception e) {
            e.printStackTrace();
        }
        new Age();
    }
}
// output:
/**
name is loading
true
sex is loading
age is loading
age construct
*/
```

> Class 仅在需要的时候才被加载， static 初始化是在类加载时就被加载。
>
> Class.forname() 取得 Class 对象引用的方法。

## Class 方法列表

| 方法名             | 介绍                                       |
| ------------------ | ------------------------------------------ |
| getName()          | Class 类型信息所存储的全限定类名           |
| isInterface()      | 是否是接口                                 |
| getSimpleName()    | Class 类型信息存储的类名(不包含包名的类名) |
| getCanonicalName() | Class 类型信息所有存储的全限定类名         |
| getInterfaces()    | 返回接口数组                               |
| getSuperClass()    | 返回直接基类                               |
| newInstance()      | 创建 class 运行时类型信息对象对应的实例    |

#### 类字面常量

使用类字面常量生成对`Class`对象的引用： 如 `FancyToy.class`；这种做法不仅简单而且安全，因为它在编译时就会受到检查（不需要放在 `try` 块中），并且根除了对 `forName()`的调用，所以高效。

类字面常量应用范围于 普通类，接口，数组，基本数据类型和包装器类， 以及一个标准字段TYPE等。TYPE字段：是一个引用，指向对应的基本数据类型的 Class对象。

```Java
boolean.class 等价于 Boolean.TYPE  
char.class 等价于 Character.TYPE  
byte.class 等价于 Byte.TYPE  
short.class 等价于 Short.TYPE  
int.class 等价于 Integer.TYPE  
long.class 等价于 Long.TYPE  
float.class 等价于 Float.TYPE  
double.class 等价于 Double.TYPE  
void.class 等价于 Void.TYPE 
```

*推荐使用 .class 的形式，以保持与普通类的一致性*

当使用  `.class` 来创建`Class`对象的引用时，不会自动初始化该 `Class` 对象。

为使用类而做的准备工作包含3个步骤。

1. **加载**。这是由类加载器执行的。该步骤将查找字节码，并从这些字节码中创建一个 Class 对象。
2. **链接**。验证类的字节码，为静态区域分配空间，如果需要的话，解析这个类创建的其他类的所有引用。
3. **初始化**。如果该类有超类，对超类初始化，执行静态初始化器 和 静态初始化代码块

```Java
class Initable {  
    static final int staticFinal = 47; // 编译期常量  
    // staticFinal2 是运行时常量，因为它需要调用其他类的方法进行赋值.  
    static final int staticFinal2 = ClassInitialization.rand.nextInt(1000);   
    static { System.out.println("Initializing Initable"); }  
}  
  
  
class Initable2 {  
    static int staticNonFinal = 147; // 非编译期常量  
    static { System.out.println("Initializing Initable2"); }  
}  
class Initable3 {  
    static int staticNonFinal = 74; // 非编译期常量  
    static {  
        System.out.println("Initializing Initable3");  
        System.out.println("staticNonFinal = " + staticNonFinal);  
    }  
}  
public class ClassInitialization {  
    public static Random rand = new Random(47);  
  
  
    public static void main(String[] args) throws Exception {  
        Class initable = Initable.class; // Initable.class  不会触发初始化  
        // 不会触发类初始化(调用常量)  
        System.out.println("Initable.staticFinal = " + Initable.staticFinal + "\n");  
        // 会触发初始化（调用常量， 但常量是有其他静态方法进行赋值的）  
        System.out.println("Initable.staticFinal2 = " + Initable.staticFinal2 + "\n");  
        // 会触发初始化（调用非final static 变量，非常数静态域）  
        System.out.println("Initable2.staticNonFinal = " + Initable2.staticNonFinal+"\n");  
          
        System.out.println("======");  
        Class initable3 = Class.forName("chapter14.Initable3"); // Class.forName() 触发类初始化  
        System.out.println("\nAfter creating Initable3 ref");  
        System.out.println("Initable3.staticNonFinal = " + Initable3.staticNonFinal); // 已经触发了Initable3初始化，不会再次触发了（仅初始化一次）  
    }  
}  
/* 
Initable.staticFinal = 47 
 
 
Initializing Initable 
Initable.staticFinal2 = 258 
 
 
Initializing Initable2 
Initable2.staticNonFinal = 147 
 
 
====== 
Initializing Initable3 
staticNonFinal = 74 
 
 
After creating Initable3 ref 
Initable3.staticNonFinal = 74 
*/  
```

#### class 对象的 cast() 方法

```Java

class Father { }  
class Child extends Father { }  
class Apple { }  
  
  
// 荔枝-class对象的cast()方法，类型转换（特别是父类转子类的荔枝）  
public class ClassCasts {  
    public static void main(String[] args) {  
        Father father = new Child();  
        Class<Child> childType = Child.class;  
  
  
        // classObj.cast(otherObj) 将 otherObj 转换为 classObj 对应的对象引用（父类对象引用转换为 子类对象引用）   
        Child child = childType.cast(father);  //    
        System.out.println(child.getClass().getName()); // chapter14.Child  
          
        child = (Child)father; // 也是将 Father 对象转换为 Child 对象（同样的效果）  
          
        // 自定义测试  
//      child = childType.cast(new Apple());  // 编译期不报错，运行时报错  
    }  
}    
// chapter14.Child  
```

## 类型转换前先检查

1. 传统的类型转换： 由RTTI 确保类型转换的正确性，若有错误抛出 ClassCastException 异常（类型转换异常）；

2. 封装对象的运行时类型信息的Class对象： 通过查询 Class 对象可以获取运行时类型信息；

3. RTTI还有第3中方式：关键字 instanceof；它返回一个 boolean值，表示对象是否属于某种类型；

  如果程序中出现了过多的 instanceof，说明程序设计存在瑕疵。

## 注册工厂

```Java
class Part {  
  public String toString() {  
    return getClass().getSimpleName();  
  }  
  static List<Factory<? extends Part>> partFactories = new ArrayList<Factory<? extends Part>>();      
  static {  
    // Collections.addAll() gives an "unchecked generic  
    // array creation ... for varargs parameter" warning.  
    partFactories.add(new FuelFilter.Factory());  
    partFactories.add(new AirFilter.Factory());  
    partFactories.add(new CabinAirFilter.Factory());  
    partFactories.add(new OilFilter.Factory());  
    partFactories.add(new FanBelt.Factory());  
    partFactories.add(new PowerSteeringBelt.Factory());  
    partFactories.add(new GeneratorBelt.Factory());  
  }  
  private static Random rand = new Random(47);  
  public static Part createRandom() {  
    int n = rand.nextInt(partFactories.size()); // partFactories.size() = 7   
    return partFactories.get(n).create();  
  }  
}     
  
  
class Filter extends Part {}  
  
  
class FuelFilter extends Filter {  
  // Create a Class Factory for each specific type:  
  public static class Factory implements typeinfo.factory.Factory<FuelFilter> { // 静态内部类作为工厂方法类  
    public FuelFilter create() { return new FuelFilter(); }  
  }  
}  
  
  
class AirFilter extends Filter {  
  public static class Factory implements typeinfo.factory.Factory<AirFilter> {// 静态内部类作为工厂方法类  
    public AirFilter create() { return new AirFilter(); }  
  }  
}     
  
  
class CabinAirFilter extends Filter {  
  public static class Factory implements typeinfo.factory.Factory<CabinAirFilter> {// 静态内部类作为工厂方法类  
    public CabinAirFilter create() { return new CabinAirFilter(); }  
  }  
}  
  
  
class OilFilter extends Filter {  
  public static class Factory implements typeinfo.factory.Factory<OilFilter> {// 静态内部类作为工厂方法类  
    public OilFilter create() { return new OilFilter(); }  
  }  
}     
  
  
class Belt extends Part {}  
  
  
class FanBelt extends Belt {  
  public static class Factory implements typeinfo.factory.Factory<FanBelt> {// 静态内部类作为工厂方法类  
    public FanBelt create() { return new FanBelt(); }  
  }  
}  
  
  
class GeneratorBelt extends Belt {  
  public static class Factory implements typeinfo.factory.Factory<GeneratorBelt> {// 静态内部类作为工厂方法类  
    public GeneratorBelt create() {  
      return new GeneratorBelt();  
    }  
  }  
}     
  
  
class PowerSteeringBelt extends Belt {  
  public static class Factory implements typeinfo.factory.Factory<PowerSteeringBelt> {// 静态内部类作为工厂方法类  
    public PowerSteeringBelt create() {  
      return new PowerSteeringBelt();  
    }  
  }  
}     
  
  
public class RegisteredFactories {  
  public static void main(String[] args) {  
    for(int i = 0; i < 10; i++)  
      System.out.println(Part.createRandom());  
  }  
}  
/*  
GeneratorBelt 
CabinAirFilter 
GeneratorBelt 
AirFilter 
PowerSteeringBelt 
CabinAirFilter 
FuelFilter 
PowerSteeringBelt 
PowerSteeringBelt 
FuelFilter 
*/  
```

## Instanceof 与 Class 的等价性

```Java
class Base {}  
class Derived extends Base {}     
// 荔枝-instanceof 与 直接比较Class对象的区别  
public class FamilyVsExactType {  
  static void test(Object x) {  
    print("class Derived extends Base {}");  
    print("x.getClass() " + x.getClass());  
    print("(x instanceof Base) " + (x instanceof Base));  
    print("(x instanceof Derived) "+ (x instanceof Derived));  
    print("Base.class.isInstance(x) "+ Base.class.isInstance(x));  
    print("Derived.class.isInstance(x) " + Derived.class.isInstance(x));  
    print("(x.getClass() == Base.class) " + (x.getClass() == Base.class));  
    print("(x.getClass() == Derived.class) " + (x.getClass() == Derived.class));  
    print("(x.getClass().equals(Base.class)) "+ (x.getClass().equals(Base.class)));  
    print("(x.getClass().equals(Derived.class)) " + (x.getClass().equals(Derived.class)));  
  }  
  public static void main(String[] args) {  
    test(new Base());  
    System.out.println("===============================");  
    test(new Derived());  
  }   
}   
/* 
============ Base 父类=================== 
class Derived extends Base {} 
x.getClass() class typeinfo.Base 
(x instanceof Base) true 
(x instanceof Derived) false 
Base.class.isInstance(x) true 
Derived.class.isInstance(x) false 
(x.getClass() == Base.class) true 
(x.getClass() == Derived.class) false 
(x.getClass().equals(Base.class)) true 
(x.getClass().equals(Derived.class)) false 
============ Derived 子类=================== 
class Derived extends Base {} 
x.getClass() class typeinfo.Derived 
(x instanceof Base) true // 子类是父类的实例 
(x instanceof Derived) true 
Base.class.isInstance(x) true // 子类是父类的实例 
Derived.class.isInstance(x) true  
(x.getClass() == Base.class) false 
(x.getClass() == Derived.class) true 
(x.getClass().equals(Base.class)) false 
(x.getClass().equals(Derived.class)) true 
*/  
```

1. instanceof 和 isInstance() 方法返回的结果完全一样，equals 和 == 的返回结果也一样
2. instanceof 和 isInstance() 保持了类型的概念，它表达的是“你是这个类吗？你是这个类的派生类吗？”
3. 而 ==比较了实际的 Class对象，就没有考虑继承，它或者是这个这个确切类型，或者不是

## 反射：运行时类信息

- 如何在运行时识别对象类型：RTTI可以告诉你对象的确切类型：前提条件，这个类在编译时类型必须被编译器知晓，这样RTTI才可以识别。
- 存在这样一种情况：在编译完成后再手动利用一串字节创建这个类的对象，即在运行时利用字节流创建对象，而不是在编译时这个对象的类型就已经知道了，如反序列化 或 远程方法调用（RMI）；这种情况下，如何知道运行时对象的确切类型呢？
- 想要获取运行时对象的类型信息的另一个动机：希望提供在跨网络的远程平台上创建和运行对象，这就是远程方法调用 RMI，RMI 允许一个java程序将对象分布到多台服务器上；
- 反射机制能够解决这个问题：即反射能够识别在运行时手动利用字节流创建的对象类型信息，即便编译器无法知道这个对象的创建；
- Class类 与 java.lang.reflect 类库一起对反射提供支持；
  - 该类库包括的类：Field、Method、Constructor（都继承自Member接口）；这些类型的对象是 jvm 在运行时创建的，用来表示未知类里的成员；
  - 如何使用这些类和方法呢？使用Constructor 创建新对象，用 get 和 set方法 读取和修改 与 Field对象相关联的字段，用invoke() 调用与 Method对相关联的方法；还有 getFields()， getMethods()， getConstuctors() 分别返回字段，方法，构造器对象数组；
- 当通过反射与一个未知类型的对象打交道时： 在使用该对象前，必须先加载该对象所属类型的Class对象；
- 因此，那个未知对象所属类的 .class 文件对于jvm来说是必须要获取的： 要么通过本地，要么通过网络；
- 反射与 RTTI的区别在于： 对于RTTI来说， 编译器在编译时打开和检查 .class 文件；而对于反射机制来说， .class文件在编译时是不可获取的，所以在运行时打开和检查 .class 文件；

## 动态代理

代理是基本的设计模式之一，代理用来代替实际对象的对象，代理充当着中间人的角色。

```Java
class ProxyTest {

    private static void consumer(Interface anInterface) {
        anInterface.doSomeThing();
        anInterface.doSomeThingElse("biu~");
    }

    public static void main(String[] args) {
        consumer(new RealObject());
        System.out.println();
        System.out.println();
        consumer(new ProxyObject(new RealObject()));
    }
}

interface Interface {
    void doSomeThing();
    void doSomeThingElse(String args);
}

class RealObject implements Interface {

    @Override
    public void doSomeThing() {
        System.out.println("doSomeThing");
    }

    @Override
    public void doSomeThingElse(String args) {
        System.out.println("doSomeThingElse:" + args);
    }
}

class ProxyObject implements Interface {

    private Interface anInterface;

    ProxyObject(Interface anInterface) {
        this.anInterface = anInterface;
    }

    @Override
    public void doSomeThing() {
        System.out.println("ProxyObject doSomeThing");
        anInterface.doSomeThing();
    }

    @Override
    public void doSomeThingElse(String args) {
        System.out.println("ProxyObject doSomeThingElse:" + args);
        anInterface.doSomeThingElse(args);
    }
}
```

代理的目的在于，任何时刻，只要你想要将额外的操作从实际对象中分离到不同地方，特别是当你希望能够做出修改时，从没有使用额外操作转为使用这些操作，或者反过来，代理就特别有用。

```Java
class DynamicProxyHandler implements InvocationHandler {

    private Object proxied;
    DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("DynamicProxy " + method.getName());
        return method.invoke(proxied, args);
    }
}

// output:
/**
doSomeThing
doSomeThingElse:biu~


ProxyObject doSomeThing
doSomeThing
ProxyObject doSomeThingElse:biu~
doSomeThingElse:biu~


DynamicProxy doSomeThing
doSomeThing
DynamicProxy doSomeThingElse
doSomeThingElse:biu~
*/
```

## 接口与类型的信息

- Interface 接口的重要目的：允许程序员隔离构件，进而降低耦合性。
- 接口并非对解耦提供了百分百的保障，因为通过类型信息，耦合性还是会传播回去。

**通过反射可以访问私有方法**

```Java
public class HiddenImplementation {  
    public static void main(String[] args) throws Exception {  
        A a = HiddenC.makeA();  
          
        a.f();  
        System.out.println("a.getClass().getName() = " + a.getClass().getName());  
        /* 这里编译报错: 找不到类表示C2 （如果C2 与 HiddenImplementation 不在同一个包下） 
         * if(a instanceof C) { C c = (C)a; c.g(); } 
         */  
          
        // 通过反射可以访问 私有方法，受保护或包可见性方法。  
        callHiddenMethod(a, "g");  
        // And even methods that are less accessible!  
        callHiddenMethod(a, "u");  
        callHiddenMethod(a, "v");  
        callHiddenMethod(a, "w");  
    }  
  
  
    static void callHiddenMethod(Object a, String methodName) throws Exception {  
        Method g = a.getClass().getDeclaredMethod(methodName); // 获取对象a的方法对象  
        g.setAccessible(true); // 设置访问权限为可访问  
        g.invoke(a); // 触发调用 对象a 的 g() 方法  
    }  
}   
/* 
public C.f() 
a.getClass().getName() = chapter14.C2 
public C.g() 
package C.u() 
protected C.v() 
private C.w() 
*/  
  
  
// C2默认为包可见性  
class C2 implements A {  
    @Override  
    public void f() {  
        print("public C.f()");  
    }  
  
  
    public void g() {  
        print("public C.g()");  
    }  
  
  
    void u() {  
        print("package C.u()");  
    }  
  
  
    protected void v() {  
        print("protected C.v()");  
    }  
  
  
    private void w() {  
        print("private C.w()");  
    }  
}  
public class HiddenC {  
    public static A makeA() {  
        return new C2();  
    }  
}    
```

- 通过使用反射，仍旧可以调用所有方法，包括private方法！如果知道方法名，就可以在其Method方法对象上调用 setAccessible(true)；
- 有些人可能认为，通过只发布编译后的代码来阻止这种情况（如外部程序调用private方法），但并不能解决问题。因为执行 javap 反编译器可以突破这个限制

**通过反射访问私有内部**

```Java
class AnonymousTest {
    static A makeA() {
        return new A() {
            @Override
            public void a() {
                System.out.println("aaaa");
            }

            @Override
            public void b() {
                System.out.println("bbbb");
            }

            @Override
            public void c() {
                System.out.println("cccc");
            }

            @Override
            public void d() {
                System.out.println("dddd");
            }
        };
    }

    private static void callHideMethod(String className, String methodName) {
        try {
            Class<?> aClass = Class.forName(className);
            Method method = aClass.getMethod(methodName);
            Object instance = aClass.newInstance();
            method.invoke(instance);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        A a = AnonymousTest.makeA();
        a.a();
        System.out.println();
        String name = a.getClass().getName();
        callHideMethod(name, "b");
        callHideMethod(name, "c");
        callHideMethod(name, "d");
    }
}

interface A {
    void a();
    void b();
    void c();
    void d();
}
```

**通过反射修改私有变量**

```Java
class WithPrivateFinalField {  
    private int i = 1;  
    private final String s = "I'm totally safe"; // 常量字符串  
    private String s2 = "Am I safe?";  
  
  
    public String toString() {  
        return "private int i = " + i + ", private final String s = " + s + ", private String s2 = " + s2;  
    }  
}  
// 荔枝-通过反射访问和修改私有变量，私有常量的荔枝  
public class ModifyingPrivateFields {  
    public static void main(String[] args) throws Exception {  
        WithPrivateFinalField pf = new WithPrivateFinalField();  
          
        System.out.println(pf + "\n");  
        // 通过反射访问私有变量i  
        Field f = pf.getClass().getDeclaredField("i");  
        f.setAccessible(true); // 设置字段 i 的可访问权限为true  
        System.out.println("Field f = pf.getClass().getDeclaredField(\"i\"); f.setAccessible(true); f.getInt(pf) = " + f.getInt(pf));  
  
  
        // 通过反射更改私有变量i  
        f.setInt(pf, 47);  
        System.out.println("f.setInt(pf, 47); f.get(pf) = " + f.get(pf));  
          
        // 通过反射访问私有常量s  
        f = pf.getClass().getDeclaredField("s");  
        f.setAccessible(true); // 设置字段 s 的可访问权限为true  
        System.out.println("f = pf.getClass().getDeclaredField(\"s\"); f.setAccessible(true); f.get(pf) 静态常量修改前 = " + f.get(pf));  
          
        // 但通过反射甚至可以修改私有常量(final) s   
        f.set(pf, "No, you're not!");  
        System.out.println("f.set(pf, \"No, you're not!\");, f.get(pf) 静态常量修改后 = " + f.get(pf));  
          
        // 通过反射访问私有变量s2  
        f = pf.getClass().getDeclaredField("s2");  
        f.setAccessible(true);  
        System.out.println("f = pf.getClass().getDeclaredField(\"s2\"); f.setAccessible(true); f.get(pf) = " + f.get(pf));  
          
        // 通过反射修改私有变量s2  
        f.set(pf, "No, you're not!");  
        System.out.println("f.set(pf, \"No, you're not!\"); f.get(pf) = " + f.get(pf));  
          
    }  
}    
/* 
private int i = 1, private final String s = I'm totally safe, private String s2 = Am I safe? 
 
 
Field f = pf.getClass().getDeclaredField("i"); f.setAccessible(true); f.getInt(pf) = 1 // 通过反射机制【访问】私有变量 
f.setInt(pf, 47); f.get(pf) = 47 // 通过反射机制 【修改】 私有变量 
f = pf.getClass().getDeclaredField("s"); f.setAccessible(true); f.get(pf) = I'm totally safe // 通过反射机制【访问】私有常量 
f.set(pf, "No, you're not!");, f.get(pf) = No, you're not! // 通过反射机制【修改】私有常量【（这个牛逼了）】 
f = pf.getClass().getDeclaredField("s2"); f.setAccessible(true); f.get(pf) = Am I safe? // 通过反射机制【访问】私有变量 
f.set(pf, "No, you're not!"); f.get(pf) = No, you're not! // 通过反射机制【修改】私有变量 
*/ 
```



## 总结

面向对象编程语言的目的：凡是可以在使用的地方都使用多态机制，只在必需的时候使用 RTTI 。

不要过早关注程序的效率问题。最好首先让程序运行起来，然后考虑他的速度。

## 感谢

[thinking-in-java（14）类型信息](https://blog.csdn.net/pacosonswjtu/article/details/78817812)