---
title: 『Thinking in Java 读书笔记』—— 9-接口
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_9
date: 1994-03-08 15:13:33

---

[Thinking in java 读书笔记](http://xiaweizi.cn/categories/Thinking-In-Java-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)

> 接口和内部类为我们提供一种将接口与实现分离的更加结构化的方法。

## 接口

`interface`关键字使抽象的概念更买进一步，`abstract`关键字允许人们在类中创建一个或者多个没有任何定义的方法，但是没有提供任何相应的具体实现，这些实现是由此类创建者创建。`interface`这个关键字产生一个完全抽象类。它根本就没有具体的实现。

`interface`不仅仅是一个极度抽象的类，因为它允许人们通过创建一个能够被向上转型为多种基类的类型，来实现某种类似多重继承变种的特性。

<!-- more -->

## 适配器

```
class StarManager implements IStar{

    IStar iStar;

    StarManager(IStar iStar) {
        this.iStar = iStar;
    }

    @Override
    public void sing() {
        System.out.println("找人唱歌");
        iStar.sing();
    }

    @Override
    public int money() {
        System.out.println("明星需要收费：" + iStar.money());
        System.out.println("代理费：" + 20);
        return iStar.money() + 10;
    }

    public static void main(String[] args) {
        StarManager manager = new StarManager(new HuangBo());
        manager.sing();
        System.out.println("一共需要花费" + manager.money());
    }
}

interface IStar {
    void sing();
    int money();
}

class HuangBo implements IStar {

    @Override
    public void sing() {
        System.out.println("黄渤 sing");
    }

    @Override
    public int money() {
        return 200;
    }
}
```

正如上方代码所示，接口的主要作用在于告诉实现类应该做什么，但是具体要做的事情还是需要导出类进行实现。

`Huangbo`作为一个`star`，自然要进行唱歌和唱歌收费，`StarManager`同样实现`IStar`接口，那就具有`IStar`同样的行为，并通过组合的方式传入`IStar`对象，起到一个代理的作用。

```
找人唱歌
黄渤 sing
明星需要收费：200
代理费：20
一共需要花费210
```

        StarManager manager = new StarManager(new HuangBo());
        manager.sing();
        System.out.println("一共需要花费" + manager.money());

通过创建`StarManager`的方式，拿到`HuangBo`对象代理，其实具体的操作在`StarManager`具体实现中实现。最终需要的`money`也就会有所变化。

## 工厂

```
class Factories {
    private static void factoryConsumer(CarFactory carFactory) {
        ICar car = carFactory.getCar();
        car.light();
        car.wheel();
    }

    public static void main(String[] args) {
        factoryConsumer(new BaoMaFactory());
        factoryConsumer(new BenChiFactory());
    }
}

interface ICar {
    void wheel();
    void light();
}

interface CarFactory {
    ICar getCar();
}

class BaoMa implements ICar {

    @Override
    public void wheel() {
        System.out.println("BaoMa--wheel");
    }

    @Override
    public void light() {
        System.out.println("BaoMa--light");
    }
}

class BenChi implements ICar {

    @Override
    public void wheel() {
        System.out.println("BenChi--wheel");
    }

    @Override
    public void light() {
        System.out.println("BenChi--light");
    }
}

class BaoMaFactory implements CarFactory {

    @Override
    public ICar getCar() {
        return new BaoMa();
    }
}

class BenChiFactory implements CarFactory {

    @Override
    public ICar getCar() {
        return new BenChi();
    }
}
```

通过接口来完成工厂模式，也是在开发的过程中经常使用的。

首先你得抽象出产品共有的行为，然后让具体的产品来实现它，比如造轮子或者汽车上面的某个零件。接下来就是建造工厂了，那工厂同样可以抽象出一个共同的特征，就是建造拥有`ICar`特征具体产品。

    ICar car = carFactory.getCar();
    car.light();
    car.wheel();

那最终就可以通过构造工厂来完成汽车的制造，而不用过多的考虑汽车具体的构造过程，同样方便后期的维护，比如需要再添加其他零件。。


