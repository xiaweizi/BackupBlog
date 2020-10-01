---
title: 『Flutter-插件篇』实现一款超酷的动态天气背景插件
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: 26ab
date: 2020-10-01 16:00:12
---

## 前言

前不久，利用周末时间学习并完成一个简单的 Flutter 项目 -  [**简悦天气**](https://github.com/xiaweizi/SimplicityWeather)，*简约不简单，丰富不复杂*，这是一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径、语音播报以及生活指数等服务，支持定位、删除、搜索等操作。

下图为主页效果，[点击下载](http://xiaweizi.top/SimplicityWeather-2_6.apk) 进行体验：

<!-- more -->

![home](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6471e071336e42e38f9037c3b5e5ae45~tplv-k3u1fbpfcp-zoom-1.image)

之前发了三篇文章：

- [『Flutter-绘制篇』实现炫酷的雨雪特效](https://juejin.im/post/6867489001809379335)
- [『Flutter-绘制篇』实现炫酷的雷电特效](https://juejin.im/post/6874061694491721736)
- [『Flutter-绘制篇』实现梦幻的晴晚流星效果](https://juejin.im/post/6871955131299168263)

对天气背景动画的具体实现做了详情的分析和总结，很多小伙伴私信我，希望可以出一个天气动态背景插件。正好当时写的时候，就以模块化的思想，单一和解耦的设计原则，放置在单独的 package 下，所以后续根据官方编写插件文档，抽成插件，希望给有需要的小伙伴能提供帮助。同时，分享如何写一个高质量插件的过程和注意事项。

因此，本篇文章的主要两个主题：

1. 介绍 [flutter_weather_bg](https://pub.flutter-io.cn/packages/flutter_weather_bg) 此插件的功能和使用说明
2. 分享编写插件的方法和注意事项

## 超酷的动态天气背景插件

### 介绍

[pub 地址](https://pub.flutter-io.cn/packages/flutter_weather_bg)

[Github 地址](https://github.com/xiaweizi/flutter_weather_bg)

这是一款丰富炫酷的天气动态背景插件，支持 **15** 种天气类型。

### 功能

- 支持 **15** 种 天气类型：晴、晴晚、多云、多云晚、阴、小中大雨、小中大雪、雾、霾、浮尘和雷暴
- 支持 动态缩放尺寸，适配多场景下展示
- 支持 切换天气类型时过度动画

### 支持的平台

- Flutter Android
- Flutter iOS
- Flutter web
- Flutter desktop

### 安装

添加 `flutter_weather_bg: ^2.8.2` 到 `pubspec.yaml` 文件中，并且导包：

```dart
import 'package:flutter_weather_bg/flutter_weather_bg.dart';
```

### 使用

通过创建 `WeatherBg` 配置天气类型，需要传入宽高来完成最终展示

```dart
WeatherBg(weatherType: _weatherType,width: _width,height: _height,)
```

### 截图效果

对不同的特点进行相应的 gif 展示。

1. 全屏沉浸式翻页效果

![home](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92cbc9dbbd19419793ffec0e9f04457b~tplv-k3u1fbpfcp-zoom-1.image)


2. 城市管理列表效果

![list](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6076f6ca3af4d3a8778313006ab9663~tplv-k3u1fbpfcp-zoom-1.image)


3. 多样的宫格效果

![grid](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2e396e151764a7e8f86e798282833b9~tplv-k3u1fbpfcp-zoom-1.image)


4. 切换天气类型时的过度动画效果

![check](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c4e75165cbb4a87b8176baa636c432d~tplv-k3u1fbpfcp-zoom-1.image)


5. 修改宽高的适配效果

![width_height](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8a4002423db4b77b0d2374f6da1c055~tplv-k3u1fbpfcp-zoom-1.image)

## 编写一款高质量的插件

目前 Flutter 那么火，离不开高热度的社区以及丰富的插件，所以如果有想法或者有能力的话，尽可能的为社区贡献出一份微薄之力吧。

那既然选择编写并发布一款插件，就要对其负责，不只是负责完成编码任务，尽可能的提供详细的使用文档说明，有条件的话，以图文并茂的方式更好，这样开发者才会愿意使用你的插件。

### 创建插件项目

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2df0281077d14679a00ab234d1abdb26~tplv-k3u1fbpfcp-zoom-1.image)

使用 Android Studio 可以直接创建 Flutter Plugin，或者直接通过命令 `flutter create -t plugin flutter_demo_plugin` 直接在当前文件夹生成插件目录。

创建好后，会生成如下的文件目录：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4c45c1030694265954cb00face29533~tplv-k3u1fbpfcp-zoom-1.image)

| 目录         | 描述                                 |
| ------------ | ------------------------------------ |
| android/     | android 端相关代码                   |
| ios/         | ios 相关代码                         |
| example/     | 样例演示项目                         |
| lib/         | flutter 端代码                       |
| test/        | 单元测试代码                         |
| CHANGELOG    | 每个版本的更新日志说明               |
| LICENSE      | license 相关                         |
| pubspec.yaml | 项目名称、项目描述、版本、依赖等相关 |
| README.md    | 项目的 readme 文档                   |

### 编写插件逻辑

#### Android 端

在 `android/src/main/java/com/example/flutter_demo_plugin/FlutterDemoPlugin.java` 中编写与 Android 端交互的逻辑。

```java
public class FlutterDemoPlugin implements FlutterPlugin, MethodCallHandler {

  private MethodChannel channel;

  @Override
  public void onAttachedToEngine(@NonNull FlutterPluginBinding flutterPluginBinding) {
    channel = new MethodChannel(flutterPluginBinding.getFlutterEngine().getDartExecutor(), "flutter_demo_plugin");
    channel.setMethodCallHandler(this);
  }

  public static void registerWith(Registrar registrar) {
    final MethodChannel channel = new MethodChannel(registrar.messenger(), "flutter_demo_plugin");
    channel.setMethodCallHandler(new FlutterDemoPlugin());
  }

  @Override
  public void onMethodCall(@NonNull MethodCall call, @NonNull Result result) {
    if (call.method.equals("getPlatformVersion")) {
      result.success("Android " + android.os.Build.VERSION.RELEASE);
    } else {
      result.notImplemented();
    }
  }

  @Override
  public void onDetachedFromEngine(@NonNull FlutterPluginBinding binding) {
    channel.setMethodCallHandler(null);
  }
}
```

1. 创建 `MethodChannel`，定义通信的唯一标识 flutter_demo_plugin
2. 注册
3. 根据方法来处理相应的逻辑

#### iOS 端

iOS 类似，只不过是按照 iOS 端的写法。

1. 创建 `FlutterDemoPlugin.h`

```objective-c
#import <Flutter/Flutter.h>

@interface FlutterDemoPlugin : NSObject<FlutterPlugin>
@end
```

2. 在 `FlutterDemoPlugin.m` 中编写逻辑

```objective-c
#import "FlutterDemoPlugin.h"

@implementation FlutterDemoPlugin
+ (void)registerWithRegistrar:(NSObject<FlutterPluginRegistrar>*)registrar {
  FlutterMethodChannel* channel = [FlutterMethodChannel
      methodChannelWithName:@"flutter_demo_plugin"
            binaryMessenger:[registrar messenger]];
  FlutterDemoPlugin* instance = [[FlutterDemoPlugin alloc] init];
  [registrar addMethodCallDelegate:instance channel:channel];
}

- (void)handleMethodCall:(FlutterMethodCall*)call result:(FlutterResult)result {
  if ([@"getPlatformVersion" isEqualToString:call.method]) {
    result([@"iOS " stringByAppendingString:[[UIDevice currentDevice] systemVersion]]);
  } else {
    result(FlutterMethodNotImplemented);
  }
}

@end
```

#### Flutter 端

直接在 `lib/flutter_demo_plugin.dart` 中调用定义好的方法即可：

```dart
class FlutterDemoPlugin {
  static const MethodChannel _channel =
      const MethodChannel('flutter_demo_plugin');

  static Future<String> get platformVersion async {
    final String version = await _channel.invokeMethod('getPlatformVersion');
    return version;
  }
}
```

### 发布前准备

代码逻辑编写完毕，离发布还有最后一步，也是编写高质量插件的重要一步。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5670d58e0f0b4e05b296bbcbda619aa5~tplv-k3u1fbpfcp-zoom-1.image)

在项目发布后，等待一段时间，这里会有对项目的评价分数，开发者可以据此作为选择的标准。那如何达到此标准，接下来会进行介绍。

1. 在 pubspec.yaml 中配置版本信息

```
// 配置项目名称
name: flutter_demo_plugin
// 配置项目描述，简要并能概括项目的内容和特点
description: A new Flutter plugin.
// 项目版本，每次发布更新，都需要增加此配置
version: 0.0.1
// 作者信息
author:
// 主页信息
homepage:

environment:
  sdk: ">=2.7.0 <3.0.0"
  flutter: ">=1.20.0 <2.0.0"

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter

// 如果只是纯 dart 插件，此断可以不加
flutter:
  plugin:
    platforms:
      android:
        package: com.example.flutter_demo_plugin
        pluginClass: FlutterDemoPlugin
      ios:
        pluginClass: FlutterDemoPlugin
      macos:
        pluginClass: FlutterDemoPlugin
      web:
        pluginClass: FlutterDemoPlugin
        fileName: flutter_demo_plugin.dart
```

2. 提供详细的 `README.md` 项目使用介绍，最好是英文

3. 提供详细的 `CHANGELOG.md` 版本迭代更新日志

4. 在 `example/lib/main.dart` 中，要有详细的代码使用案例

5. 代码中，对公有方法和变量，提供尽可能详细的说明

6. 完善插件支持平台

7. 代码保证完整性、正确性和可编译

8. 插件包中的所有依赖要保证是最新版本

这些是通过脚本工具自动检测，除此静态的配置，更重要的代码质量和尽可能的去维护，这样才是一款合格的插件。

### 发布

最后，只需要通过 ` flutter packages pub publish --server=https://pub.dartlang.org` 命令即可发布。

当看到：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b831968ed4a04d7fa467df4660e72bcb~tplv-k3u1fbpfcp-zoom-1.image)

恭喜证明你上传成功了


