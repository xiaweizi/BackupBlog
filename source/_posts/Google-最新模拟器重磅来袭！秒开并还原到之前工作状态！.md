---
title: Google-最新模拟器重磅来袭！秒开并还原到之前工作状态！
date: '2017.12.27 20:46:25'
categories:
  - 技术分享
tags:
  - Google 官方
abbrlink: 6515
---


## 前言

12月18日，Google 官方[Quick Boot](https://android-developers.googleblog.com/2017/12/quick-boot-top-features-in-android.html)博客的发布，给我们带来了最新的`Android`模拟器，其中最突出的特点技术 **快速启动**。声称可以在 **6** 秒之内便可启动模拟器，在此之下，模拟器通过保存关闭之前的快照，实现数秒内便可恢复到之前的工作状态。

<!-- more -->

废话不多说，来看一下效果：

![Quick Boot.gif](http://upload-images.jianshu.io/upload_images/4043475-e0b50be455409f79.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

正好之前写了个小需求 [自定义跑马灯](https://www.jianshu.com/p/fdf460359857)，效果就很明显。

在关闭模拟器时，绿色的跑马灯停止在「跑」字位置，经过短暂的保存状态过程，再次启动模拟器，你会发现不到 **1s** 中模拟器变运行起来，并且跑马灯接着「跑」字继续滚动。

## 主要特点

除了 **Quick Boot** 强大的功能之外，[Quick Boot](https://android-developers.googleblog.com/2017/12/quick-boot-top-features-in-android.html) 这篇博客还强调一些最近发布的功能。其实 `Google` 从两年前 [Android Studio 2.0 Preview: Android Emulator](https://android-developers.googleblog.com/2015/12/android-studio-20-preview-android.html) 模拟器发布以来，都一直致力于提过速度和稳定性，并增加一系列丰富的功能用来加速开发者的应用开发和测试。跟随者此次的更新，绝对值得将 `Android` 模拟器升级到最新的版本！

### 快速启动

此次，作为一项稳定版本的发布，**快速启动** 是你的模拟器在 **6s** 之内便可恢复之前的状态。首次启动 `Android` 模拟器时，还是得必须像之前启动设备那样的冷启动，但是后续的速度便会加快，系统会恢复到关闭之前的状态，类似于唤醒设备。`Google` 通过彻底对模拟器系统的重构完成此次功能，并处理了虚拟传感器和 `GPU` 加速。从 `Android` 模拟器 `v 27.0.2` 开始，默认情况下启用 **Quick Boot**，因此是不需要额外的配置的。

### 兼容性

从 `v4.4` 到最新的每个 `SDK` 版本，`Google` 都会确保模拟器能够满足开发人员的日常需求。不过为了提高模拟器系统镜像的品种和稳定性，现针对 `Android Nougat` (API24) 及其以上做了限制要求。

### Google Play 支持

在国内对 `Google Play` 的需求不是很多，但是在国外，很多开发者还是会用到 `Google Play` 服务，在之前的模拟器中，要想保持最新的服务还是很困难的。为了解决这个问题，从 `API24` 开始，`Google` 提过了包含其服务的系统镜像版本，可以正常的使用`Google` 服务，就像是在真机上一样。

### 性能改进

使用模拟器 **快速、高效** 的开发一直是`Google`团队持续目标，在过去的时间里，不断研究模拟器开发的性能影响，特别是内存使用情况。使用最新版本的 `Android` 模拟器，可以根据需要分配内存，而不是根据在`AVD`中设置的固定值来分配。

此外，在过去的几个版本中，还改进了`CPU`和`I/O`的性能，增强了`GPU`的性能，包括`OpenGL ES 3.0` 的支持。从一种图片可以看出改进的效果：

![adb Push 比较.png](http://upload-images.jianshu.io/upload_images/4043475-ceba0742ebe1f24d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


对于`GPU`性能方面，`Google`创建了[GPU仿真模拟压力测试](https://github.com/google/gpu-emulation-stress-test)程序来根据时间进行衡量。我们发现最新的模拟器相比较之前提高了不少的帧率，同时它也是模拟器中极少部分能根据`Android`规范准确的呈现`OpenGL ES 3.0`.

![GPU.gif](http://upload-images.jianshu.io/upload_images/4043475-a9dcb3f9f903702f.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**GPU 仿真压力测试：**

![stressTest.png](http://upload-images.jianshu.io/upload_images/4043475-3ea1a759f4da9615.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 更多功能

还有一些去年添加的功能，防止不知道先列出来：

- **WI-FI 支持**  从 `API24`开始，可以创建虚拟的蜂窝网络或者是`WI-FI`。
- **Google Cast 支持**  当你使用`Google Play`系统镜像时，在同一个`WI-FI`下可以将屏幕投射到`Chromcast` 设备上。
- **拖拽 APK和文件**  通过拖动`APK`文件到模拟器上，便可实现快速安装；也可以直接拖拽文件到模拟器上，并在模拟器的`DownLoad` 文件夹中找到它。
- **本地复制和粘贴**  可以在本地和模拟器直接复制粘贴文本
- **两个手指的动作**  在使用谷歌地图时，按住`ctrl`(Windows、Linux)或者`⌘`(Mac)，并用鼠标即可实现缩放或放大效果。
- **模拟GPS位置**
- **虚拟传感器** 在扩展控制面板中有一个专门的界面，支持`Android`模拟器中的传感器，包括加速，旋转等
- **WebCam 的支持** 可以使用网络摄像头或者笔记本电脑内置摄像头作为`AVD`中的虚拟相机，在管理器的 **高级设置** 页面中确认相机设置。
- **本地键盘** 可以使用本地外设键盘进行内容输入
- **虚拟短信和电话呼叫**
- **屏幕缩放**
- **窗口大小缩放**
- **网络代理支持**  到 *代理* 选项下的设置界面，为模拟器添加自定义`HTTP`代理。
- **错误报告**  可以使用扩展面板中的错误报告快速生成应用程序的错误报告，和团队分享或者向`Google`反馈。

![setting.png](http://upload-images.jianshu.io/upload_images/4043475-8be5e21e8528ac6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 获取

![EmulatorGet.png](http://upload-images.jianshu.io/upload_images/4043475-7e52c52cfbdf736e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所有的功能和改进都可以通过将图中`Android Emulator`更新到 v27.0.2+ 获取。 


## 小bug

不知道你们有没有遇到，我在使用的过程中，模拟器黑屏的时候，会出现怎么都打不开的现象，无论重启还是按模拟器的电源键都没有效果。然后按照网上的方法，尝试着改了一下`RAM`，任意改成与之前不同的值就可以了。应该是因为修改了系统属性导致重新加载才能恢复正常吧。

![peizhi.png](http://upload-images.jianshu.io/upload_images/4043475-390f451c2275ec5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

----

> 以上所有的内容和部分图片全部来自官方博客：[Quick Boot](https://android-developers.googleblog.com/2017/12/quick-boot-top-features-in-android.html)

**感谢！！**
