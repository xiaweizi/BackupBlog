---
title: Thinking-in-Java 读书笔记-11-持有对象
author: 下位子
tags:
  - Thinking In Java
  - 读书笔记
categories:
  - Thinking In Java 读书笔记
abbrlink: thinking_in_java_11
date: 1994-03-10 15:13:33
---

> 如果一个程序只包含固定数量的且其生命期都是已知的对象，那么这是一个非常简单的程序。

通常，程序总是根据运行时才知道的某些条件去创建新对象，在此之前，不会知道所需对象的数量，甚至不知道确切的类型。为解决这个普遍的编程问题，需要在任意时刻和任意位置创建任意数量的对象。所以，就不能依靠创建命名的引用来持有每一个对象。

<!-- more -->

## 泛型和类型安全容器

在使用`Java SE5`之前的容器是允许你向容器中插入不正确的类型，为了避免这种错误，在之后对类型进行严格的编译判断。只有相同的类型或者直属导出类才能创建容器。通过使用泛型，就可以在**编译器**防止将错误类型的对象放置在容器中。



## 基本概念



`Java`容器类类库的用途是“保存对象”，并将其划分为两个不同的概念：

1. **Collection。**一个独立对象的序列，这些元素都服从一条或多条规则。**List**必须按照插入的顺序保存元素，而 **Set** 不能有重复元素。**Queue**按照排队规则来确定对象产生的顺序(通常与它们被插入的顺序相同)。
2. **Map。**一组成对的“键值对”对象，允许你使用键来查找值，因此在某种意义上讲，它将数字与对象关联在一起。**映射表**允许我们使用另一对象来查找某个对象，它也被称为“关联数组”，因为他将某些对象与另外一些对象关联在一起了。**Map** 是强大的编程工具。

## 容器的打印

通过代码来看一下效果:

```java
class PrintContainers {

    static Collection fill(Collection<String> collection) {
        collection.add("rat");
        collection.add("cat");
        collection.add("dog");
        collection.add("dog");
        return collection;
    }

    static Map fill(Map<String, String> map) {
        map.put("rat", "Fuzzy");
        map.put("cat", "Rags");
        map.put("dog", "Bosco");
        map.put("dog", "Spot");
        return map;
    }

    public static void main(String[] args) {
        System.out.println(fill(new ArrayList<String>()));
        System.out.println(fill(new LinkedList<String>()));
        System.out.println(fill(new HashSet<String>()));
        System.out.println(fill(new TreeSet<String>()));
        System.out.println(fill(new LinkedHashSet<String>()));
        System.out.println(fill(new HashMap<String, String>()));
        System.out.println(fill(new TreeMap<String, String>()));
        System.out.println(fill(new LinkedHashMap<String, String>()));
    }
}
```

输出结果:

```java
[rat, cat, dog, dog]
[rat, cat, dog, dog]
[rat, cat, dog]
[cat, dog, rat]
[rat, cat, dog]
{rat=Fuzzy, cat=Rags, dog=Spot}
{cat=Rags, dog=Spot, rat=Fuzzy}
{rat=Fuzzy, cat=Rags, dog=Spot}
```

## List

有两种 **List**类型的容器：

- **ArrayList。**它擅长于随机访问元素，但是在 **List** 中间插入数据和移除元素时比较慢。
- **LinkedList。**它通过代价低的在 **List** 中间进行插入和删除操作，提供了优化的顺序访问。 **LinkedList**在随机访问方面相对比较慢，但是它的特性集较 **ArrayList** 更大。

## 迭代器

**迭代器**是一个对象，它的工作是遍历并选择序列中的对象，而客户端程序不必知道或关心该序列底层的结构。此外迭代器通常被称为 **轻量级对象**：创建它的代价小。

`Java`的 **Iterator** 只能单向移动，这个 **Iterator** 只能用来：

1. 使用方法 `iterator()`要求容器返回一个 `Iterator`。
2. 使用 `next()`获得序列中的下一个元素。
3. 使用 `hasNext()` 检查序列中是否还有元素。
4. 使用 `remove()` 将迭代器新近返回的元素删除。

```java
class IteratorTest {
    public static void main(String[] args) {
        List<Integer> data = new ArrayList<Integer>();
        data.add(1);
        data.add(2);
        data.add(3);
        data.add(4);
        data.add(5);

        ListIterator<Integer> integerListIterator = data.listIterator(2);
        while (integerListIterator.hasNext()) {
            Integer next = integerListIterator.next();
            System.out.println("next:" + next);
        }

        while (integerListIterator.hasPrevious()) {
            Integer previous = integerListIterator.previous();
            System.out.println("previous:" + previous);
        }
    }

    private static void iterator() {
        List<Integer> data = new ArrayList<Integer>();
        data.add(1);
        data.add(2);
        data.add(3);
        data.add(4);
        Iterator<Integer> iterator = data.iterator();
        while (iterator.hasNext()) {
            Integer next = iterator.next();
            System.out.println("data: " + next);
        }

        iterator = data.iterator();
        for (int i = 0; i < 4; i++) {
            iterator.next();
            iterator.remove();
        }
        System.out.println(data);
    }
}
```

## Set

`Set` 不保存重复的元素（至于如何判断元素相同则较为复杂）。如果你视图将相同对象的多个实例添加到 `Set`中，那么它就会阻止这种重复现象。

```
class SetTest {
    public static void main(String[] args) {
        Set<Integer> data = new HashSet<Integer>();
        data.add(1);
        data.add(2);
        data.add(3);
        data.remove(4);
        System.out.println(data.remove(1));
        System.out.println(data);
    }
}
```

## Map

将对象映射到其他对象的能力是一种解决编程问题的杀手锏。

```Java
class MapTest {
    public static void main(String[] args) {
        Map<Integer, String> data = new HashMap<Integer, String>();
        data.put(2, "2");
        data.put(1, "1");
        data.put(3, "3");

        System.out.println(data);
    }
}
```

## 总结

`Java` 提供了大量持有对象的方式：

1. 数组将数字与对象联系起来。它保存类型明确的对象，查询对象时，不需要对结果做类型转换。它可以是多维的，可以保存基本类型的数据。但是，数组一旦生成，其容量就不能改变。
2. `Collection` 保存单一的元素，而 `Map` 保存相关联的键值对。有了 `Java` 的泛型，你就可以指定容器中存放的对象类型，因此你就不会讲错误类型的对象放置到容器中，并且在从容器中获取元素时，不必进行类型转换。
3. 像数组一样， `List` 也建立数字索引与对象的关联，因此，数组和 `List` 都是排好序的容器。 `List` 能够自动扩充容量。
4. 如果进行大量的随机访问，就是用 `ArrayList`；如果经常从表中插入或删除元素，则应该使用 `LinkedList`。
5. 各种 `Queue` 以及栈的行为，由 `LinkedList`提供支持。
6. `Map`是一种将对象与对象相关联的设计。 `HashMap` 设计用来快速访问，而 `TreeMap` 保持 键 始终处于排序状态，所以没有 `HashMap` 快。 `LinkedHashMap` 保持元素插入的顺序，但是也通过散列提供了快速访问能力。
7. `Set`不接受重复元素。 `HashSet` 提供最快的查询速度，而 `TreeSet` 保持元素处于排序状态。 `LinkedHashSet` 以插入顺序保存元素。
8. 新程序中不应该使用过时的 `Vector`、 `Hashtable`和 `Stack`。

