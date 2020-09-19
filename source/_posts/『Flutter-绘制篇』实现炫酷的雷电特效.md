---
title: 『Flutter-绘制篇』实现炫酷的雷电特效
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: 5c93
date: 2020-09-19 12:11:12
---


## 前言

前不久，利用周末时间学习并完成一个简单的 Flutter 项目 -  [**简悦天气**](https://github.com/xiaweizi/SimplicityWeather)，*简约不简单，丰富不复杂*，这是一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径、语音播报以及生活指数等服务，支持定位、删除、搜索等操作。

下图为主页效果，[点击下载](http://xiaweizi.top/SimplicityWeather-2_3.apk) 进行体验：

![图1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76bdbd8781d64fe49f253207020b53b4~tplv-k3u1fbpfcp-zoom-1.image)

<!-- more -->

项目中运用了大量的自定义绘制 widget，首页丰富的 [自定义 chart](https://juejin.im/post/6868920194568781838) 效果和炫酷的天气背景动效。天气背景动效在不同的天气气象下展示不同的效果。目前一共实现了 **15** 种类别，其中有，晴、晴晚、多云、多云晚、阴天、小中大雨、小中大雪、雾、霾、浮尘以及雷暴。背景动效一共分为三层：

- 背景颜色层。从上到下的渐变效果
- 云层。只有一种图片，对其位移、数量、染色做不同变化达到不同效果
- 信息层。包括雨雪、雷暴和晴晚流星效果

之前分别用两篇文章介绍雨雪和晴晚流星效果的实现细节：

- [『Flutter-绘制篇』实现炫酷的雨雪特效](https://juejin.im/post/6867489001809379335)
- [『Flutter-绘制篇』实现梦幻的晴晚流星效果](https://juejin.im/post/6871955131299168263)

今天我们介绍背景动画的最后一篇，如何实现炫酷的雷电特效，先看一下最终效果：

![thunder](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/687120e980df40a198eadc97bbc4b44c~tplv-k3u1fbpfcp-zoom-1.image)

## 准备

根据实现效果进行分析，雨滴效果在之前文章有介绍过不多赘述，仔细观察，其实就是对闪电图片在绘制时控制其 alpha 以营造出这种霹雳的效果。

首先准备几张闪电的素材，UI 网站找了很长时间没有找到满意的效果，关键费时费钱。后来发现 oppo 最新版的天气的雷暴效果停酷炫的，于是对其反编译，找到他的资源目录。其实 oppo 雷暴的动画效果是通过 视频+openGL 的方式实现，里面有闪电的静态资源、视频资源和 openGL 代码文件。我们只需提取他的静态资源文件即可。随即在 initState() 方法中异步获取加载图片资源：

```dart
  Future<void> fetchImages() async {
    weatherPrint("开始获取雷暴图片");
    var image1 = await ImageUtils.getImage('assets/images/lightning/lightning0.webp');
    var image2 = await ImageUtils.getImage('assets/images/lightning/lightning1.webp');
    var image3 = await ImageUtils.getImage('assets/images/lightning/lightning2.webp');
    var image4 = await ImageUtils.getImage('assets/images/lightning/lightning3.webp');
    var image5 = await ImageUtils.getImage('assets/images/lightning/lightning4.webp');
    _images.add(image1);
    _images.add(image2);
    _images.add(image3);
    _images.add(image4);
    _images.add(image5);
    weatherPrint("获取雷暴图片成功： ${_images?.length}");
  }
```

有了图片后，开始构建对象和参数列表。由上面分析可知，除了基本的坐标 x,y 信息，只需要额外增加 alpha 属性来达到效果。

```dart
class ThunderParams {
  ui.Image image;
  double x;
  double y;
  double alpha;
  int get imgWidth => image.width;
  int get imgHeight => image.height;

  ThunderParams(this.image);

  void reset() {
    x = Random().nextDouble() * 0.5.wp -  1 / 3 * imgWidth;
    y = Random().nextDouble() * -0.05.hp;
    alpha = 0;
  }
}
```

`reset()`  方法用于在当前雷暴结束时，重新初始化参数信息。

## 绘制

参数配置好后，绘制很简单。有了图片有了位置信息和 alpha 信息，调用 canvas 的相关 api 进行绘制即可。

```dart
  void drawThunder(ThunderParams params, Canvas canvas, Size size) {
    if (params == null || params.image == null) {
      return;
    }
    canvas.save();
    var identity = ColorFilter.matrix(<double>[
      1, 0, 0, 0, 0,
      0, 1, 0, 0, 0,
      0, 0, 1, 0, 0,
      0, 0, 0, params.alpha, 0,
    ]);
    _paint.colorFilter = identity;
    canvas.drawImage(params.image, Offset(params.x, params.y), _paint);
    canvas.restore();
  }
```
绘制到屏幕中大概长这样:

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e893b7064eb9463b88f07d3d79fdd4e2~tplv-k3u1fbpfcp-zoom-1.image" alt="thunder2" style="zoom: 33%;" />

## 动画

离炫酷就差最后一步 **动画**。

首先我们把单个闪电看做一个动画对象，从消失到展示再显示，落实到动画上，alpha 由0到1，然后再到0。但是你可能发现，出现的速度要比消失的速度要快。我们可以借助 TweenSequence 类来实现这个效果。

TweenSequence 是一个动画序列，支持配置权重，以及对应的动画 Tween。这样，我们可以给 alpha 在 [0,1] 区间做动画时权重设置低一点，[1,0] 时权重高一点。

```dart
    var _animation = TweenSequence([
      TweenSequenceItem(
          tween: Tween(begin: 0.0, end: 1.0)
              .chain(CurveTween(curve: Curves.easeIn)),
          weight: 1),
      TweenSequenceItem(
          tween: Tween(begin: 1.0, end: 0.0)
              .chain(CurveTween(curve: Curves.easeIn)),
          weight: 3),
    ]);
```

实现后，效果如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b71751fa89d24300ad7956ee7ed5e98a~tplv-k3u1fbpfcp-zoom-1.image)

然后，我们用三个随机的闪电作为一组，做循环动画，控制其序列帧，完成连续&不同&随机的删掉效果。

```dart
	_controller = AnimationController(duration: Duration(seconds: 1), vsync: this);
    _controller.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        _controller.reset();
        Future.delayed(Duration(milliseconds: 10)).then((value) {
          initThunderParams();
          _controller.forward();
        });
      }
    });

    var _animation = TweenSequence([
      TweenSequenceItem(
          tween: Tween(begin: 0.0, end: 1.0)
              .chain(CurveTween(curve: Curves.easeIn)),
          weight: 1),
      TweenSequenceItem(
          tween: Tween(begin: 1.0, end: 0.0)
              .chain(CurveTween(curve: Curves.easeIn)),
          weight: 3),
    ]).animate(CurvedAnimation(
      parent: _controller,
      curve: Interval(
        0.0, 1.0,
        curve: Curves.ease,
      ),
    ));
```

在之前说的一个闪电动画后面，新增 `.animate()` 的配置，通过控制 

```dar
Interval(
        0.0, 0.3,
        curve: Curves.ease,
      )
```

配置该动画执行序列帧的开始和结束，以及插值器。

通过在 `addStatusListener` 中监听动画的执行状态，在触发 `AnimationStatus.completed` 时随机等待一定时间后，重新开始。

到此，一个炫酷的雷电特效就完成了，是不是很简单，如果觉得还不错，后面考虑把天气动画背景做成插件供有需要的小伙伴使用

