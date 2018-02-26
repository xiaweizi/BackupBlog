title: 使用Kotlin重构项目
date: 2017.09.09 20:46:25
categories:
- 技术分享
tags:
- Kotlin
- 项目重构
---

## 前言

上周大概花了一个星期的时间初步学习了一下`Kotlin`，并且同步写了[Kotlin 笔记](http://www.jianshu.com/p/f132e368b88d)，方便后面使用的时候查询一些语法的用法。

<!-- more -->

一周的`Kotlin`学习下来，虽然只掌握了`Kotlin`的皮毛，但仍被其简单便捷的语法吸引。目前`Kotlin`已经成为`Android`的官方推荐语言，所以建议有时间的同学不妨学习一下，相信一定可以帮助你提高开发效率的。

这里就不介绍`Kotlin`的语法使用，既然初步学习了`Kotlin`，那么就在实践中检验一下，使用的是之前的毕业设计项目

<div class="github-widget" data-repo="xiaweizi/QNews"></div>

，这个项目总体来说很适合初学者，所以就拿它上手，改起来相对比较简单。

所以本篇文章就介绍一下`Kotlin`在`Android`中的使用，相信你会爱上他的。

先看一下运行效果吧，跟之前[快毕业了，撸个小项目](http://www.jianshu.com/p/ae4aa11f35a4)效果是一模一样，所以大致看一下即可：

![运行效果.gif](http://upload-images.jianshu.io/upload_images/4043475-65effeb19b751e39.gif?imageMogr2/auto-orient/strip)

这里就展示一下基本的效果，想要了解详情的就`fork`一下我的代码吧:[QNews](https://github.com/xiaweizi/QNews)

## 使用步骤

### 1. `Android Studio`集成`Kotlin`插件

通过`Android Studio`添加`Kotlin`插件：

![Kotlin插件](http://upload-images.jianshu.io/upload_images/4043475-f78dbad6f4cb211f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进行相关的配置：


![相关配置](http://upload-images.jianshu.io/upload_images/4043475-eed2a897793f5265.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

集成好了，记得同步一下，这个时候在`build.gradle`下会生成对`Kotlin`插件的引用：

    // 根 build.gradle
    classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    
    // module build.gradle
    
    apply plugin: 'kotlin-android'
    compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"

这个时候配合着[anko](https://github.com/Kotlin/anko)使用效果更好呦，只需要在`build.gradle`中进行相应的配置：

    apply plugin: 'kotlin-android-extensions'
    compile "org.jetbrains.anko:anko-sdk15:0.9.1"

就这么两行的配置即可告别`findViewById`的繁琐过程，目前这个项目对于`ButterKnife`的依赖我都去掉了，极其方便。

只要`XML`定义了`"@+id/test"`，就可以直接引用这个`View`.

比如我的这个跳转界面`SplashActivity`的`XML`：

    <ImageView
        android:id="@+id/iv_splash"
        android:background="?attr/colorPrimary"
        android:layout_centerInParent="true"
        android:src="@mipmap/ic_splash"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

代码中可以直接使用(属性动画)：

    val translationX: ObjectAnimator = ObjectAnimator.ofFloat(iv_splash, "translationX", 600f, 0f)
