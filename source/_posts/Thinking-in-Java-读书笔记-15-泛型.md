---
title: 『Thinking in Java 读书笔记』—— 15-泛型
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_15
date: 1994-04-15 17:13:33

---

## 简单泛型

泛型的主要目的之一就是用来指定容器要持有什么类型的对象，而且由编译器你来保证类型的正确性，Java 泛型的核心概念，告诉编译器想使用什么类型，然后编译器帮你处理一切细节。

**元组类库**

比如要想实现一个元组(它是将一组对象直接打包存储存于其中的一个单一对象)类库，实现如下：

<!-- more -->

```Java
public class TwoTuple<A, B> {
    public final A first;
    public final B second;
    public TwoTuple(A a, B b) {
        first = a;
        second = b;
    }
    public String toString() {
        return "(" + first + ", " + second + ")";
    }
}
```

客户端程序可以读取`first`或`second`所引用的对象，然后可以随心所欲地使用这两个对象。但是，它们却无法将其他值赋予`first`或`second`。因为`final`声明为你买了安全保险，实现了`Java`编程的安全性原则，而且这种格式更加简洁明了。

**堆栈类**

```Java
public class LinkedStack<T> {
    private static class Node<U> {
        U item;
        Node<U> next;
        Node() {
            item = null;
            next = null;
        }
        Node(U item, Node<U> next) {
            this.item = item;
            this.next = next;
        }
        boolean end() {
            return item == null && next == null;
        }
    }
     
    private Node<T> top = new Node<T>();
    public void push(T item) {
        top = new Node<T>(item, top);
    }
    public T pop() {
        T result = top.item;
        if (!top.end()) {
            top = top.next;
        }
        return result;
    }
    public static void main(String[] args) {
        LinkedStack<String>  lss = new LinkedStack<>();
        for (String s : "aaa bbb ccc".split(" ")) {
            lss.push(s);
        }
        String temp;
        while ((temp = lss.pop()) != null) {
            System.out.println(temp);
        }
    }
}
// output:
/**
ccc
bbb
aaa
*/
```

## 泛型方法

1. 可以在类中包含参数化方法，而这个方法所在的类可以是泛型类，也可以不是泛型类。也就是说，是否拥有泛型方法，与其所在的类是否是泛型没有关系。
2. 如果使用泛型方法可以取代将整个类泛型化，那么就应该只使用泛型化，另外对于一个static方法而言，无法访问泛型类的类型参数，所以如果static方法需要使用泛型能力，就必须使其成为泛型方法。
3. 要定义泛型方法，只需将泛型参数列表置于返回值之前。

```Java
public <T> void f(T x) {
    System.out.println(x.getClass().getName());
}
```

如果调用**f()**时传入的是基本类型，自动打包机制就会介入其中，将基本类型的值包装为对应的对象。

#### 可变参数与泛型方法

泛型方法与可变参数列表能够很好的共存

```Java
public static <T> List<T> makeList(T... args) {
    List<T> result = new ArrayList<T>();
    for (T items : args) {
        result.add(item);
    }
    return result;
}
```

#### 简化元组的使用

我们现在可以重新编写之前的元组工具，使其成为更通用的工具类库。

```Java
public class Tuple {
    public static <A, B> TwoTuple<A, B> tuple(A a, B b) {
        return new TwoTuple<A, B>(a, b);
    }
    ......
	static TwoTuple<String, Integer> f() {
        return Tuple.tuple("hi", 47);
    }
    static TwoTuple f2() {
        return Tuple.tuple("hi", 47);
    }
    public static void main(String... args) {
        TwoTuple<String, Integer> ttsi = f();
        System.out.println(f());
        System.out.println(f2());
    }
}
/*
Output:
(hi, 47)
(hi, 47)
*/
```

## 擦除的神秘之处

```Java
public class Test {
    public static void main() {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        if (c1 == c2)
            System.out.println("true");
        System.out.println(Arrays.toString(c1.getTypeParameters()));
        System.out.println(Arrays.toString(c2.getTypeParameters()));
    }
}
/*
Output:
true
[E]
[E]
*/
```

1. **ArrayList<String>**和**ArrayList<Integer>**很容易被误认为是两种不同的类型，但它们是相同的。首先我们可以看到，上面的程序会认为它们是相同的类型。另外，程序中使用的**Class.getTypeParameters()**可以返回一个**TypeVariable**对象数组，表示有泛型声明所声明的类型参数，但是我们能够发现的只有用作参数占位符的标识符。因此我们得出结论：**在泛型代码内部，无法获得任何有关泛型参数类型的信息。**


2. Java泛型是使用擦除来实现的，这意味着**当你在使用泛型时，任何具体的类型信息都被擦除了**，你唯一知道的就是你在使用一个对象。因此**List<String>**和**List<Integer>**在运行时都被擦除成它们的“原生”类型**List**。
3. 由于擦除，**在使用泛型时，这些类型参数都会被当作Object进行处理**。

## 擦除的补偿

擦除丢失了在泛型代码中执行某些操作的能力。**任何在运行时需要知道的确切类型信息的操作都将无法工作**。

```Java
public class Erased<T> {
    private final int SIZE = 100;
    public static void f(Object arg) {
        if (arg instanceof T) {)                    //ERROR
        T var = new T();                             //ERROR
        T[] array = new T[SIZE];                //ERROR
        T[] array = (T) new Object[SIZE];  //ERROR
    }
}
```

**创建类型实例**

对于上面这个问题Java的解决方法是使用工厂对象来创建新的实例，建议使用显式工厂，并限制其类型，使得只能接受实现了这个工厂的类。

```Java
interface Factory<T> {
    T create();
}

class Foo<T> {
    private T x;
    public <F extends Factory<T>> Foo(F Factory) {
        x = factory.create();
    }
}

class IntegerFactory implements Factory<Integer> {
    public Integer create() {
        return new Integer(0);
    }  
}

class Widget {
    public static class WidgetFactory implements Factory<Widget> {
        public WidgetFactory create() {
            return new Widget();
        }
    }
}

public class FactoryConstraint {
    public static void main(String... args) {
        new Foo<Integer>(new IntegerFactory());
        new Foo<Widget>(new Widget.WidgetFactory());
    }
}
```

## 边界

1.边界使得我们可以在用于泛型的类型参数类型上设置限制条件。因为擦除了类型信息，所以可以用无界泛型参数调用的方法只是那些可以用 `Object`调用的方法。但是，如果能够将这个参数限制为某个类型子集，那就可以用类型子集来调用方法。

2.为了执行对泛型参数的限制，`Java`重用了`extends`关键字。

3.对泛型进行参数限制也有多继承，并且也可以通过继承消除冗余。

```Java
interface A{}

interface B {}

class C{}

class D extends C implements A, B {}

class E<T extends C & A & B> {}

class F {
    public F() {
        E<D> e = new E<D>();
    }
}
```

4.下面的程序展示了如何在继承的每个层次上添加边界限制。

```Java
class A {
   void set();
}

class B<T> {
   T item;
   public B(T item) {
       this.item = item;
   }
}

class C<T extends A> extends B<T> {
   T item;
   public C(T item) {
       this,item = item;
       item.set();
   }
}
```

## 通配符

数组具有**协变性**：可以向导出类型的数组赋予基类型的数组引用。

```java
class Fruit {}
class Apple extends Fruit {}
class Jonathan extends Apple {}
class Orange extends Fruit {}

class CovariantArrays {
    public static void main(String... args) {
        Fruit[] fruit = new Apple[10];
        fruit[0] = new Apple(); //OK
        fruit[1] = new Jonathan(); //OK
        fruit[2] = new Orange(); //error
    }
}
```

上面的代码不会出现编译问题，因为`Apple`、`Orange`、`Jonathan`都是`Fruit`的子类型，`Fruit`类型的引用持有它们并没有任何问题，是有意义的。但是`fruit[2] = new Orange();`这一句在运行时会抛出`ArrayStoreException`异常，因为数组fruit在运行时的实际类型为`Apple`。

**数组的协变性对List并不起作用**。

```Java
List<? extends Fruit> flist = new ArrayList<Apple>();
//Compile Error: can't add any type of object
//flist.add(new Apple());
//flist.add(new Fruit);
//flist.add(new Object());
```

上面代码中唯一的限制就是这个`List`要持有某种具体的`Fruit`或`Fruit`的子类型，但是编译器实际上并不知道`List`持有什么类型，那么也就不能安全地向其中添加对象，因此会出现编译时错误。

虽然在上面的程序中`List`的`add()`方法不可用，但是并不是所有的方法都是不可用的。

```Java
public class CompilerIntelligence {
    public static main(String... args) {
        List<? extends Fruit> flist = Arrays.asList(new Apple());
        Apple a = (Apple) flist.get(0);
        flist.contains(new Apple());
        flist.indexOf(new Apple());
    }
}
```

这两段程序的区别在于`add()`方法将接受一个具有泛型参数类型的参数，但是`contains()`和`indexOf()`将返回或者接受`Object`类型的参数。因此，在指定一个`ArrayList<? extends Fruit>`时，`add()`的参数就变成了"`? extends Fruit`"，此时，编译器并不能了解这里需要`Fruit`的哪个具体子类型，因此它不会接受任何类型的`Fruit`。而另外的方法使用了`Object`，并不涉及通配符，因此编译器也将允许这个调用。而上面的`get()`方法只会也只能返回`Fruit`对象，这是在该泛型参数所给定了边界——“任何扩展自`Fruit`的对象”之后所能做的唯一的事情了。

**逆变**

超类通配符：使用方法是由某个特定类的任何基类来界定的即**<? super MyClass>**

```Java
List<? super Apple> apples = Arrays.asList(new Apple());
apples.add(new Apple());
apples.add(new Jonathan());
apples.add(new Fruit()); //Error
```

**无界通配符**

1. 第一种情况下**无界通配符**意味着“任何事物”，即编译器很少关心使用的是原生类型还是**<?>**。因此，**<?>**是在声明：**我是想用Java的泛型来编写代码，我在这里并不是要用原生类型，但是在当前这种情况下，泛型参数可以持有任何类型**。
2. 泛型的另一种应用是：**当你在处理多个参数时，优势允许一个参数可以时任喝类型，同时为其他参数确定某种特定类型**，如`Map<String, ?> map = new HashMap<String, Integer>`。
3. **List**实际表示“持有任何**Object**类型的**List**”，而**List<?>**表示“具有**某种特定类型**的非原生**List**，只是我们不知道那种类型是什么。”
4. 使用确切类型来代替通配符，可以使用泛型参数来做更多的事，但是使用通配符使得你必须接受范围更宽的参数化类型作为参数。

一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会成为相同的接口。

使用带有泛型参数类型的转型或**instanceof**不会带有任何效果。

## 自限定的类型

1.自限定：

```Java
class SelfBounded<T extends SelfBounded<T>> {}
```

**SelfBounded**类接受泛型参数**T**，而**T**由一个边界限定，这个边界就是拥有**T**作为其参数的**SelfBounded**。
2.自限定强制泛型当作自己的边界参数来使用。如：

```Java
class A extends SelfBounded<A> {}
```

3.自限定限制只能强制作用于继承关系。如果食用自限定，就应该了解这个类所用的类型参数将与使用这个参数的类具有相同的基类型。这会强制要求使用这个类的每个人都要遵循这种形式。

## 动态类型安全

1.Java SE5中的**java.util.Collections**中有一组工具用于检查容器所持有的类型是否是我们所需要的，它们是：**静态方法checkedCollection()**、**checkedList()**、**checkedMap()**、**checkedSet()**、**checkedSortedMap()**、**checkedSortedSet()**。这些方法会将你希望动态检查的容器当作第一个参数接受，并将你希望强制要求的类型作为第二个参数接受。

```Java
List<Dog> dogs = Collections.checkedList(new ArrayList<Dog>(), Dog.class);
```

2.因为可以向Java SE5之前的代码传递泛型容器，所以旧式代码仍旧有可能破坏你的容器，此时上述工具就可以解决在这种情况下的类型检查问题。

## 混型

1.混型的最基本的概念是混合多个类的能力。以产生一个可以表示混型中所有与类型的类。

**一.与接口混合**

1.一种常见的产生混型效果的方法是使用接口：

```Java
interface TimeStamped {
    long getStamp();
}

class TimeStampedImp implements TimeStamped {
    private final long timeStamp;
    public TimeStampedImp() {
        timeStamp = new Date().getTime();
    }
    public long getStamp() {
        return timeStamp;
    }
}

interface SerialNumbered {
    long getSerialNumbered();
}

class SerialNumberedImp implements SerialNumbered {
    private static long counter = 1;
    private final long serialNumber = counter++;
    private long getSerialNumber() {
        return serialNumber;
    } 
}

interface Basic {
    public void set(String val);
    public String get();
}

class BasicImp implements Basic {
    private String value;
    public void set(String val) {
        this.value = val;
    }
    public String get() {
        return value;
    }
}

class Mixmin extends BasicImp implements TimeStamped, SerialNumbered {
    private TimeStamped timeStamp = new TimeStampedImp();
    private SerialNumbered serialNumber = new SerialNumberedImp();
    public long getStamp() {
        return timeStamp.getStamp();
    }
    public long getSerialNumber() {
        return serialNumber.getSerialNumber();
    }
}

public class Mixmins {
    public static void main (String... args) {
        Mixmin mixmin1 = new Mixmin(), mixmin2 = new Mixmin();
        mixmin1.set("string1");
        mixmin2.set("string1");
        System.out.println(mixmin1.get() + "  " + mixmin1.getStamp() + "  " + mixmin1.getSerialNumber());
        System.out.println(mixmin2.get() + "  " + mixmin2.getStamp() + "  " + mixmin2.getSerialNumber());
    }
}
```

这个示例的使用方法非常简单，但是当使用更加复杂的混型时，代码量会急速增加。

**二.使用装饰器模式** 

1.装饰器是通过使用组合和形式化结构来实现的，而混型时基于继承的。因此可以将基于参数化类型的混型当作一种泛型装饰器机制，这种机制不需要装饰器设计模式的继承结构：

```Java
class Basic {
    private String value;
    public void set(String val) {
        this.value = val;
    }
    public String get() {
        return value;
    }
}

class Decorator extends Basic {
    protected Basic basic;
    public Decorator(Basic basic) {
        this.basic = basic;
    }
    public void set(String val) {
        basic.set(val);
    }
    public String get() {
        return basic.get();
    }
}

class TimeStamped extends Decorator {
    private final long timeStamp;
    public TimeStampedImp(Basic basic) {
        super(basic);
        timeStamp = new Date().getTime();
    }
    public long getStamp() {
        return timeStamp;
    }
}

class SerialNumbered extends Decorator {
    private static long counter = 1;
    private final long serialNumber = counter++;
    public SerialNumbered(Basic basic) {
        super(basic);
    }
    private long getSerialNumber() {
        return serialNumber;
    } 
}

public class Decoration {
    public static void main(String...args) {
        TimeStamped t = new TimeStamped(new Basic());
        TimeStamped t2 = new TimeStamped(new SerialNumbered(new Basic()));

        SerialNumbered t = new SerialNumbered(new Basic());
        SerialNumbered t2 = new SerialNumbered(new TimeStamped(new Basic()));
    } 
}
```

对于装饰器来说，其明显的缺陷谁它只能有效地工作于装饰中的最后一层，而混型方法显然会更佳自然一些，因此，装饰器只是对由混型提出的问题的一种局限的解决方案。

**三.与动态代理结合** 

1.可以使用动态代理来创建一种比装饰器更贴近混型模型的机制。由于动态代理的限制，每个被混入的类都必须时某个接口的实现：

```Java
class MixminProxy implements InvocationHandler {
    Map<String, Object> delegatesByMethod;
    public MixminProxy(TwoTuple<Object, Class<?>>... pairs) {
        delegatesByMethod = new HashMap<String, Object>();
        for (TwoTuple<Object, Class<?>> pair : pairs) {
            for (Method method : pair.second.getMethods()) {
                String methodName = method.getName();
                if (!delegatesByMethod.containsKey(methodName)) 
                    delegatesByMethod.put(methodName, pair.first);
            }
        }
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        Object delegate = delegatesByMethod.get(methodName);
        return method.invoke(delegate, args);
    }

    public static Object newInstance(TwoTuple... pairs) {
        Class[] interfaces = new Class[pairs.length];
        for (int i = 0; i < pairs.length; i ++) {
            interfaces[i] = (Class) pairs[i].second;
        }
        ClassLoader cl = pair[0].first.getClassLoader();
        return Proxy.newProxyInstance(cl, interfaces, new MixminProxy(pairs));
    }
}

public class DynamicProxyMixmin {
    public static void main(String... args) {
        Object mixmin = MixminProxy.newInstance(
            tuple(new BasicImp(), Basic.class), 
            tuple(new TimeStampedImp(), TimeStamped.class), 
            tuple(new SerialNumberedImp(), SerialNumbered.class));
        Basic b = (Basic) mixmin;
        TimeStamped t = (TimeStamped) mixmin;
        SerialNumbered s = (SerialNumbered) mixmin;
        b.set("Hello");
        System.out.println(b.get());
        System.out.println(t.get());
        System.out.println(s.get());
    }
}
```

这种方案要比上面两种方式更加接近于真正的混型。

## 潜在类型机制

1.某些编程语言提供了一种机制——**潜在类型机制**。
2.泛型代码典型地将在泛型类型上调用少量方法，而具有潜在类型机制的语言只要求实现某个方法的子集，而不是某个特定类或接口，从而放松了这种限制。
3.潜在类型机制是一种代码组织和复用机制。有了它编写出来的代码相对于没有它编写出的代码，能够更容易滴复用。
4.由于泛型是后期才加进Java的，因此没有任何机会可以去实现任何类型的潜在类型机制，因此Java没有对这种类型的支持。

## 对缺乏潜在类型机制的补偿

##### 一.反射

在Java中使用潜在类型机制，可以使用的一种方式是反射，下面的**perform()**方法就是用了潜在类型机制：

```
class Mime {
    public void walkAgainstTheWind() {}
    public void sit() { print("Pretending to sit"); }
    public void pushInvisibleWalls() {}
    public String toString() { return "Mime"; }
}

class SmartDog {
    public void speak() { print("Woof!"); }
    public void sit() { print("Sittint"); }
    public void reproduce() {}
}

class CommunicateReflectively {
    public static void perform(Object speaker) {
        Class<?> spkr = speaker.getClass();
        try {
            try {
                Method speak = spkr.getMethod("speak");
                speak.invoke(speaker);
            } catch (NoSuchMethodException e) {
                print(speaker + "cannot speak");
            }
            try {
                Method sit = spkr.getMethod("sit");
                sit.invoke(sit);
            } catch (NoSuchMethodException e) {
                print(speaker + "cannot sit");
            }
        } catch (Exception e) {
            throw new RuntimeException(speaker.toString(), e);
        }
    }
}

public class LatentReflection {
    public static void main(String.... args) {
        CommunicateReflectively.perform(new SmartDog());
        CommunicateReflectively.perform(new Robot());
        CommunicateReflectively.perform(new Mime());
    }
}
```

上述代码中，这些类都是彼此分离的。

**将一个方法应用于序列**

1.反射虽然提供了潜在类型机制的可能性，但是它将所有类型检查都转移到了运行时。如果能够实现编译期类型检查，这通常会更加符合要求。
2.假设想要创建一个方法，它能够将任何方法应用于某个系列中的所有对象。我们可以使用上面的反射以及可变参数来解决这个问题：

```Java
class Shape {
    public void rotate() { print(this + " rotate") }
    public void resize(int newSize) {
        print(this + " resize" + newSize);
    }
}

class Square extends Shape {}

class Apply {
    public static <T, S extends Iterable<? extends T>> void apply (S seq, Method f, Object... args) {
        try {
            for (T t : seq) {
                f.invoke(t, args);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

class FilledList<T> extends ArrayList<T> {
    public FilledLIst(Class<? extends T> type, int size) {
        try {
            for (int i = 0; i < size; i ++) {
                add(type.newInstance());
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

class Test {
    public static void main(String... args) throws Exception {
        List<Shape> shapes = new ArrayList<Shape>();
        for (int i = 0; i < 10; i ++)
            shapes.add(new Shape());
        Apply.apply(shapes, Shape.class.getMethod("rotate"));
        Apply.apply(shapes, Shape.class.getMethod("resize", int.class), 5);

        Apply.apply(new FilledList<Shape>(Shape.class, 10), Shape.class.getMethod("rotate"));
    }
}
```

尽管之中方法的解决方法背证明很优雅， 但是我们必须知道使用反射比非反射可能要慢一些，因为动作都是在运行时发生的。

**三.当你并未碰巧拥有正确的接口时**

1.如果具有潜在类型机制的参数化类型机制，你不会受任何特定类库的创建者过去所作的设计的支配，不想上面的代码需要适合需求的接口，因此这样的代码不是特别的“泛化”。

**四.用适配器模仿潜在类型机制**

1.实际上，潜在类型机制创建一个包含所需方法的*隐式接口*。因此它遵循这样的规则L如果我们手工编写了必需的接口，那么它就应该能够解决问题。
2.从我们拥有的接口中编写代码来产生我们需要的接口，这是适配器设计模式的一个典型示例。我们可以使用适配器来适配已有的接口，以产生想要的接口。

## 感谢

[《Thinking in Java》学习](https://www.jianshu.com/p/adf709de8001)