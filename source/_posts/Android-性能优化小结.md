---
title: Android-性能优化小结
date: '2017.08.08 20:46:25'
categories:
  - 技术分享
tags:
  - 性能优化
abbrlink: 9202
---

> 本周有个需求，对某个界面进行优化，然后看了一些文章，并进行小结，为了方便以后回头查看。

![](http://upload-images.jianshu.io/upload_images/4043475-7c357364ccfb7dbd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!-- more -->

*仅供个人参考*

![我还可以继续战斗](http://upload-images.jianshu.io/upload_images/4043475-323934e1385bfad3.gif?imageMogr2/auto-orient/strip)


### 一、界面绘制优化
#### 1. 页面卡顿原因
- 布局 Layout 过于复杂，无法在16 ms 内完成渲染。
- 同一时间动画执行的次数过多，导致 CPU 或 GPU 负载过重
- View 过度绘制，导致某些像素在同一帧时间内被绘制多次
- UI 线程中做了稍微耗时的操作

#### 2. 解决工具
- 开发者选项-打开GPU渲染
- 使用 `Systrace`
- 使用 `TraceView`
- 使用 `Hierarchy Viewer` 观察每个 View 的绘制时间

#### 3. 解决策略
- 如果布局层数比较多的时候，推荐使用 RelativeLayout
- 如果布局嵌套比较多，推荐使用 LinearLayout (RelativeLayout 的view的排列方式是基于彼此于彼此依赖)
- 使用 include 标签进行布局复用
- 使用 merge 标签去除多余层级
	- merge 标签最好是替代 FrameLayout 或者是布局一致的 LinearLayout，比如当前布局的 LinearLayout 是垂直方向的，被包含的布局的 LinearLayout 也是垂直方向的则可以使用 merge 标签。
- 使用 ViewStub 延迟加载布局提高加载速度
	- ViewStub 只能加载一次，加载后 ViewStub 被置为空 
	- ViewStub 不能嵌套 merge 标签
	- ViewStub 操作的是布局文件，如果想操作具体的 view，还是要使用 view 的visible 属性
- 避免过度绘制、重绘
	- 移除不需要的 background
	- 在自定义 view 的 onDraw 方法中，用 canvas.clipRect 来指定绘制的区域，放置重叠的组件发生过度绘制

 
### 二、内存泄露
#### 1. 主要原因分类
1. 开发人员自己编写代码造成的内存泄漏
2. 第三方框架造成的泄漏
3. 由 Android 系统或者第三方 ROM 造成的内存泄漏

#### 2. 泄露场景

- 非静态内部类的静态实例
	- 如果非静态内部类里面创建的实例是静态的，那么它会间接的长期维持着外部的引用，阻止被系统回收
- 匿名内部类的静态实例
- Handler 的内存泄露
	- 如果 Handler 是非静态的，那么 Handler 也会导致引用它的 Activity、Service or Fragment，导致内存泄露
- 未正确使用 Context (最常见的就是单例模式)
- 静态的 view
- 资源对象未关闭(curson file)
- 集合中对象未清理
- Bitmap 对象
	- 避免静态变量持有比较大的 btimap 对象或者其他大的数据对象，如果已持有，要尽快置空静态变量
- 监听器未关闭
	- 很多系统的服务 TelephonyManager SensorManager 记得取消注册，或者注册 null



## 三、开发过程中遵循的守则
### 1. 编程思想
应用层的性能优化通常可以从以下几个方面考虑：

- 了解编程语言的编译原理，使用高效编码方式从语法上提高程序性能；
- 采用合理的数据结构和算法提高程序性能，这往往是决定程序性能的关键；
- 重视界面布局优化；
- 采用多线程、缓存数据、延迟加载、提前加载等手段，解决严重的性能瓶颈；
- 合理配置虚拟机堆内存使用上限和使用率，减少垃圾回收频率；
- 合理使用native代码；
- 合理配置数据库缓存类型和优化SQL语句加快读取速度，使用事务加快写入速度；
- 使用工具分析性能问题，找出性能瓶颈；


### 2. 编程技巧

**不执行不必要的操作(CPU)、不分配不必要的内存(内存)**

- 避免创建不必要的对象
	- 分配内存本身需要时间，虚拟机运行时堆内存使用量是有上限的，容易出发 GC，使进程暂停，造成严重卡顿。
- 合理使用 static 成员
- 不需要操作运行时的动态变量和方法，那么可以讲方法设置为 static
- 常量字段要声明为 static final，因为常量会被放在 dex 文件的静态字段初始化器中被直接访问。否在运行时会通过编译器自动生成一些函数来初始化此规则只对基本类型和 String 类型有效
- 避免创建 static 的 view or context，持有 Activity 的引用容易造成内存泄露
- 避免内部的 getter 和 setter
	- 尽量直接声明为 public，直接访问的速度比间接访问要快7倍
- 合理使用浮点类型
	- 在 Android 设备中，浮点型大概比整形数据处理速度慢两倍
- 移除 Activity 默认背景，提升 Activity 加载速度
	- `getWindow().setBackgroundDrawable(null);`
- cursor stream 的关闭
- 广播的注册和取消注册
- 合理使用 `StringBuffer` `StringBuilder` `String`
- 尽量使用局部变量
- 调用方法时传递的参数已经调用中创建的临时变量都保存在 stack 中，速度较快，其他变量(静态变量、成员变量)都在 heap 中创建，速度较慢
- IntentService 代替 Service
- 使用 `ApplicationCotext ` 代替 Activity 的 Context
- 集合中的对象及时清理
- 记得在 `onDestory()` 方法中，及时销毁 webView
