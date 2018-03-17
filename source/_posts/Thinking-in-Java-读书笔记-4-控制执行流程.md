---
title: Thinking-in-Java-读书笔记-4-控制执行流程
date: '1994.02.28 18:27:25'
categories:
  - Thinking In Java 读书笔记
tags:
  - Thinking In Java
  - 读书笔记
abbrlink: thinking_in_java_4
---

[Thinking in java 读书笔记](http://xiaweizi.cn/categories/Thinking-In-Java-%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)

就像有知觉的生物一样，程序必须在执行过程中控制它的世界，并做出选择。在 Java 中，你要使用执行控制语句来做出选择。**

作为一名程序员，尤其经常接触业务需求的开发人员，那流程的接触是必不可少的，基本的用法也是很熟练，所以在这就不介绍流程的基础知识了，直接看代码。

<!-- more -->

## if else

```
private static void checkGrade(int grade) {
        if (grade >= 90) {
            System.out.println("优秀");
        } else if (grade >= 80) {
            System.out.println("良好");
        } else if (grade >= 70) {
            System.out.println("中等");
        } else if (grade >= 60) {
            System.out.println("及格");
        } else {
            System.out.println("不及格");
        }
    }
```

## while

```
    private static void testWhile() {
        int i = 1;
        int sum = 0;
        while (i <= 10) {
            sum += i;
            i++;
        }
        do {
            sum += i;
            i++;
        } while (i <= 10);
        System.out.println("sum:" + sum);
    }
```

## for

```
    private static void testFor() {
        for (int i = 1; i < 20; i++) {
            if (i % 2 == 0) {
                System.out.println(i + "是偶数");
                continue;
            }
            System.out.println(i + "是奇数");
        }
    }
```

## switch

```
    private static void testSwitch() {
        int sex = 0;
        switch (sex) {
            case 0:
                System.out.println("男性");
                break;
            case 1:
                System.out.println("男性");
                break;
            default:
                System.out.println("未知");
                break;
        }
    }
```