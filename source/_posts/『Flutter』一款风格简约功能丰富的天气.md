---
title: 『Flutter』一款风格简约功能丰富的天气
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: f7f8
date: 2020-08-29 19:11:12
---

### 前言

最近利用两周的周末和下班时间入坑了 Flutter，简单的学了一下基础的 widget 后，就准备拿一个项目练练手。什么项目既实用，有不乏技术，当时是写一个天气 app 了，因此  [**简悦天气**](https://github.com/xiaweizi/SimplicityWeather) 因此诞生。

*简约不简单，丰富不复杂*

一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径以及生活指数等服务，支持定位、删除、搜索、语音播报等操作。

作为 flutter 实战项目，包含状态管理、网络请求、数据缓存、自定义 view、自定义动画，三方统计，事件管理等技术点，实用且丰富。

<!-- more -->

### 体验

点击[下载链接](http://xiaweizi.top/SimplicityWeather-2_2.apk)下载

或者直接扫描二维码抢先体验

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d6af5e8c51b40799b47776130f86d62~tplv-k3u1fbpfcp-zoom-1.image)


### 功能介绍

目前已支持的功能如下

- [x] 自动定位
- [x] 添加&删除城市
- [x] 实时信息
- [x] 24小时&多日预报
- [x] 丰富的生活指数
- [x] 台风路径
- [x] 背景高斯模糊
- [x] 动态降雨卡片
- [x] 自动更新
- [x] 丰富多样的天气背景效果(雷暴效果)
- [x] 自动升级
- [x] 语音播报
- [x] **一键换天**，做天气之子

接下来多图警告
![图1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76bdbd8781d64fe49f253207020b53b4~tplv-k3u1fbpfcp-zoom-1.image)

![图2](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/671df9cddce54ed59fc5617cc082ec0d~tplv-k3u1fbpfcp-zoom-1.image)

![图3](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5dd1014c00f447d88a5ea287786e808~tplv-k3u1fbpfcp-zoom-1.image)

#### 一键换天

天气背景效果分为三层：

- 背景颜色层。从上到下的渐变效果
- 云层。只有一种图片，对其位移、数量、染色做不同变化达到不同效果
- 雨雪层。为雨雪天气单独做了动画，很炫酷。

目前支持多达 **12** 种不同的天气类型，其中包括：晴、多云、阴天、小中大雨、小中大雪、雾、霾、浮尘，为了更好，在关于页面有上角添加切换天气类型的入口，实时查看不同气象下不同的背景效果。下面用一种 GIF 图展示效果，鉴于 GIF 本身的局限，可能会模糊低帧，请下载 apk 自行体验。

![图4](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9b3ed513c8348ddbbdc0a538038dfdd~tplv-k3u1fbpfcp-zoom-1.image)

![图5](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d86dcaaafb0b4e7ba05be8fbe0bb84e4~tplv-k3u1fbpfcp-zoom-1.image)

![图6](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/252455b3876d4788bf7eec42eea63282~tplv-k3u1fbpfcp-zoom-1.image)

### 技术介绍

虽然这个项目简单，但是涵盖的 flutter 技术点却不少，可以满足日常开发小项目的需求。

其实从入门 flutter 到完成这个项目，也只是花了两个周末，加上平时下班的时间，代码写的略微有点仓促，很多可以封装，和一些绘制的逻辑还有很大的优化空间，后面抽时间进行优化一波。

大致整理一下其技术点：

- 网络请求&数据缓存
- 高德 SDK 以及自动定位
- 自定义 View，动画以及 widget
- 三方统计接入，插件接入
- 状态管理，数据同页面以及跨页面传输
- 屏幕适配，公共组件封装

这里特地把自定义 View &动画拉出来说说，从项目可以看到很多的图表，包括空气质量&湿度比例圆弧图，日出日落弧线图，以及24小时&多日折线图。其实绘制原理跟 Android 的绘制很像，包括接口的设计也异曲同工。同样有 canvas 画布和 paint 画笔，这样写起来就方便很多。

有了 Android 绘制基础，写起来就非常快了，这几个图加在一起一共不到一天的时间就可以完成。无非是先装载好数据，根据获取到的数据进行绘制。后面有时间会单独介绍一下这这块的绘制逻辑。

### 三方库依赖

Fluter 为什么目前这么火热，以及开发的效率如此之高，除了因为 Google 这个大佬作为靠山，更多的还是活跃的开源社区，[pub.dev](https://pub.flutter-io.cn/) 上提供了很多稳定而又多样的插件，省去不必要的开发时间，根据自己的实际情况进行选择，结合 Mac 上 Alfred 工具，搜索起来得心应手。

同样，简悦天气项目中也引用不少依赖插件，特此进行说明和感谢。

| 插件名称                                                     | 插件说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [flutter_bloc](https://pub.flutter-io.cn/packages/flutter_bloc) | 不多说，很强大，作为新手再也不用担心数据共享与更新           |
| [shared_preferences](https://pub.flutter-io.cn/packages/shared_preferences) | 很常用，持久化保存数据                                       |
| [dio](https://pub.flutter-io.cn/packages/dio)                | 网络请求当然离不开一个好的网络框架，项目只是用到了基础的 GET 请求 |
| [amap_location_fluttify](https://pub.flutter-io.cn/packages/amap_location_fluttify) | 高德定位插件，虽然有坑，但是满足基础需求                     |
| [location_permissions](https://pub.flutter-io.cn/packages/location_permissions) | 定位权限判断已经申请插件                                     |
| [event_bus](https://pub.flutter-io.cn/packages/event_bus)    | 事件更新                                                     |
| [flutter_slidable](https://pub.flutter-io.cn/packages/flutter_slidable) | 强大的侧滑插件                                               |
| [umeng_analytics_plugin](https://pub.flutter-io.cn/packages/umeng_analytics_plugin) | 友盟统计插件                                                 |
| [flutter_screenutil](https://pub.flutter-io.cn/packages/flutter_screenutil) | 屏幕适配工具插件                                             |
| [modal_bottom_sheet](https://pub.flutter-io.cn/packages/modal_bottom_sheet) | 底部弹窗                                                     |
| [path_drawing](https://pub.flutter-io.cn/packages/path_drawing) | 绘制虚线 path 用到                                           |
| [url_launcher](https://pub.flutter-io.cn/packages/url_launcher) | 通用跳转工具                                                 |
| [package_info](https://pub.flutter-io.cn/packages/package_info) | 获取包相关信息                                               |
| [ota_update](https://pub.flutter-io.cn/packages/ota_update)  | apk 自动升级                                                 |
| [flutter_tts](https://pub.flutter-io.cn/packages/flutter_tts) | 语音播报                                                     |

### 总结

对 Flutter 两周的学习和开发下来后的感受，代码写起来是真快，在 Android 如果想实现，点击按钮并更新 TextView 的逻辑，首先创建 xml 布局，其次 java 代码中初始化 View 并给 button 添加点击事件，然后，还需要有单独的变量保存状态的更新，最后在变量发生变化时更新 TextView 的值。

而 Flutter 的核心就是  widget，很多功能都封装在 widget 中，在 Flutter 中如果想实现该功能，只需创建两个 widget，在相应 click 事件时只需关系变量值改变，调用 setState，后续的 View 更新可以完全不用管，有点类似 Android 的 databinding。

不过，由于全部都是 widget，你会发现如果把所有页面写在一个文件，那么嵌套的画面绝对会让你崩溃。这就需要，对功能进行拆解，尽量做到逻辑最小化，代码阅读性和维护性也会很高。还有就是插件，虽然社区的插件已经相当丰富，但还是有些需求没有找到合适的插件，这就需要广大开发者，积极燃烧开源热血，为社区贡献自己的成果。

总的来说，开发效率上确实提高不少，因为入坑时间短，原理和性能上没有细致研究，所以可能导致看法有点片面，不过后面会努力补上知识点。

### 感谢

天气数据来源于 [彩云科技](https://open.caiyunapp.com/Main_Page)

定位功能来自 [高德地图](https://lbs.amap.com/) 

> 高德 appkey 放在代码中，供大家使用，结果有人恶意刷 api，导致我账号被封，所以部分功能不能正常使用，请见谅

统计来自 [友盟统计](https://mobile.umeng.com/)

天气 icon 来自 [阿里icon](https://www.iconfont.cn/)

### ChangeLog

#### 2.2.0

- 新增 OTA 时更新说明
- 晴天新增太阳
- 支持语音播报功能
- 容错高德 API 次数限制处理

#### 2.1.0

- 新增自动更新
- 新增雷暴效果
- 新增动态降雨卡片
- 更换 iOS icon
- 修复删除城市后背景不更新问题

#### 2.0.0

- 优化雨雪绘制算法，提升雨雪类型的性能
- 新增雪花随风摇摆效果

#### 1.7.0

- 新增高斯模糊效果

#### 1.6.0

- 第一次提交