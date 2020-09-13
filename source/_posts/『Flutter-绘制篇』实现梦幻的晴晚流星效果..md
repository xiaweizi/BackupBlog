---
title: 『Flutter-绘制篇』实现梦幻的晴晚流星效果
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: 903e
date: 2020-09-13 23:11:12
---


### 前言

前不久，利用周末时间学习并完成一个简单的 Flutter 项目 -  [**简悦天气**](https://github.com/xiaweizi/SimplicityWeather)，*简约不简单，丰富不复杂*，这是一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径、语音播报以及生活指数等服务，支持定位、删除、搜索等操作。

<!-- more -->

下图为主页效果，[点击下载](http://xiaweizi.top/SimplicityWeather-2_2.apk) 进行体验：

![图1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76bdbd8781d64fe49f253207020b53b4~tplv-k3u1fbpfcp-zoom-1.image)

项目中运用了大量的自定义绘制 widget，首页丰富的 [自定义 chart](https://juejin.im/post/6868920194568781838) 效果和炫酷的天气背景动效。天气背景动效在不同的天气气象下展示不同的效果。目前一共实现了 **14** 种类别，其中有，晴、晴晚、多云、阴天、小中大雨、小中大雪、雾、霾、浮尘以及雷暴。背景动效一共分为三层：

- 背景颜色层。从上到下的渐变效果
- 云层。只有一种图片，对其位移、数量、染色做不同变化达到不同效果
- 信息层。包括雨雪、雷暴和晴晚流星效果

之前在 [『Flutter-绘制篇』实现炫酷的雨雪特效](https://juejin.im/post/6867489001809379335)  一文中介绍过雨雪的实现细节，今天我们聊一聊如何实现 **梦幻的晴晚流星** 效果，其实实现到不难，主要是实现的思路和方法。

话不多说，先看一下动态效果图(为了更好预览多样的背景动效，在右上角关于页面加了入口，可以实时切换天气类型，查看动态效果，感兴趣的下载体验)：

![night_star](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bb41f0e67ad48a594c5d1ff05d0256a~tplv-k3u1fbpfcp-zoom-1.image)

> 陪你去看流星雨落在这地球上，让你的泪落在我肩膀，在我肩膀~

仔细观察不难发现，主要有两部分组成：
- 不断闪烁的星星效果
- 转瞬即逝的流星效果

所以接下来会围绕上面两部分进行详细讲解。

### 晴晚

Flutter 同 Android 的绘制有很多相似之处，提供了 canvas 和 paint 类作为画板和画笔，可以绘制基础的图形文字和图片，同样有很多 api 为简单的图形添加渐变、高斯模糊等炫酷的特效 。

#### 初始化素材和参数

晴晚由群星组成，首先绘制一颗星星，简单实现，就不使用图片，直接通过 canvas 绘制。Flutter 提供了 `MaskFilter.blur(_style, _sigma)`，并通过 `_paint.maskFilter`  可以图形设置模糊，从而营造星星的发光效果。

`BlurStyle` 一共有四种类别，分别为：normal、solid、outer和outter，对应下面的效果：
![blur_style](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7919a358f0d3487fa76e73b2b48a299c~tplv-k3u1fbpfcp-zoom-1.image)

`_sigma` 模糊系数越大越模糊，在 normal 下分别设置 0、1、3、7 效果：
![sigma](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97cb518af0ff42f68be64de6210a0fcc~tplv-k3u1fbpfcp-zoom-1.image)

有了星星，接下来就需要创建星星，并赋予其属性，让其展示在屏幕上。

```dart
class _StarParam {
  double x;
  double y;
  double alpha = 0.0;
  double scale;

  _StarParam();

  void init() {
    alpha = Random().nextDouble();
    scale = Random().nextDouble() * 0.1 + 0.6;
    x = Random().nextDouble() * 1.wp / scale;
    y = Random().nextDouble() * 0.3.hp / scale;
  }
}
```
除了坐标 x,y 属性外，alpha 用于后面做动画， scale 用于模拟远近的效果，我们创建 100 个随机分布在 `1*width` 和 `0.3*height` 的区域内。

#### 绘制

有参数后，直接开始绘制，只需要调用 `canvas.drawCircle()` 即可：

```dart
  void drawStar(_StarParam param, Canvas canvas) {
    if (param == null) {
      return;
    }
    canvas.save();
    var identity = ColorFilter.matrix(<double>[
      1, 0, 0, 0, 0,
      0, 1, 0, 0, 0,
      0, 0, 1, 0, 0,
      0, 0, 0, param.alpha, 0,
    ]);
    _paint.colorFilter = identity;
    canvas.scale(param.scale);
    canvas.drawCircle(Offset(param.x, param.y), 3, _paint);
    canvas.restore();
  }
```
实现效果如下：

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ecfffdaaa744b15bf93a4816ab25b84~tplv-k3u1fbpfcp-zoom-1.image)

#### 动画

接下来就是让星星“动”起来。有点类似之前实现雨雪的逻辑，创建一个持续运行的动画，在动画回调中不断的 `setState()`，对于星星对象，用一个属性 `alpha` 字段，不断自增或自减来达到闪烁的效果。

```dart
  void move() {
    if (reverse == true) {
      alpha -= 0.01;
      if (alpha < 0) {
        reset();
      }
    } else {
      alpha += 0.01;
      if (alpha > 1.2) {
        reverse = true;
      }
    }
  }
```
这里，当 alpha 值达到阈值时，将 reverse 字段置为 true，此时开始自减，做消失动画，因为很多属性都是随机，最终就营造出群星闪烁的效果。

![stars](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a35b063da062421499630668e7a210e5~tplv-k3u1fbpfcp-zoom-1.image)

### 流星

#### 初始化素材和参数

一开始准备从各种图片素材网上试着找一下流星的资源，结果要么效果不满意，要么就是要各种收费和关注等等。转念一想，还不如自己实现来的快，无非就是一条**渐变细长形**的**圆角**矩形。

**圆角**矩形通过 `canvas.drawRRect()` 可以实现，为了实现流星的拖尾效果，借用 `paint#shader` 的属性，达到渐变的效果。

```dart
  var gradient = ui.Gradient.linear(
    const Offset(0, 0),
    Offset(_meteorWidth, 0),
    <Color>[const Color(0xFFFFFFFF), const Color(0x00FFFFFF)],
  );
  _meteorPaint.shader = gradient;
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc6a1b777806422089a61f4c11341d92~tplv-k3u1fbpfcp-zoom-1.image)

我们随机创建4组流星。

#### 绘制

为了营造更真实的流星划过效果，水平效果是不满足的，需要对其进行角度翻转。

一开始，打算通过 y/x=tan(Θ) 方式进行绘制，后来考虑到要做动画，各种动态计算，干脆直接使用 `canvas.rotate()` 和 `canvas.translate()` 的方法，更加方便，更加的好理解。

```dart
  void drawMeteor(_MeteorParam param, Canvas canvas) {
    canvas.save();
    var gradient = ui.Gradient.linear(
      const Offset(0, 0),
      Offset(_meteorWidth, 0),
      <Color>[const Color(0xFFFFFFFF), const Color(0x00FFFFFF)],
    );
    _meteorPaint.shader = gradient;
    canvas.rotate(pi * param.radians);
    canvas.translate(param.translateX, tan(pi * 0.1) *_meteorWidth + param.translateY);
    canvas.drawRRect(
        RRect.fromLTRBAndCorners(0, 0, _meteorWidth, _meteorHeight,
            topLeft: _radius,
            topRight: _radius,
            bottomRight: _radius,
            bottomLeft: _radius),
        _meteorPaint);
    canvas.restore();
  }
```

这里 rotate 和 translate 的顺序不能写反，需要先旋转再平移，因为旋转的点在左侧端，如果先平移再旋转，会导致运动过程中，流星不再一条直线上运行。效果如下：

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/955128b04e484163b2f47f295724f9d3~tplv-k3u1fbpfcp-zoom-1.image)

#### 动画

复用晴晚的 AnimationController,通过不断的改变 translateX 值，来完成划过的动画效果。

```dart
class _MeteorParam {
  double translateX;
  double translateY;
  double radians;
  void reset() {
    translateX = 1.0.wp + Random().nextDouble() * 20.0.wp;
    radians = -Random().nextDouble() * 0.07 - 0.05;
    translateY = Random().nextDouble() * 0.5.hp;
  }

  void move() {
    translateX -= 20;
    if (translateX <= -1.0.wp) {
      reset();
    }
  }
}
```
这里初始化 `1.0.wp + Random().nextDouble() * 20.0.wp`，使其初始化位置分布在 [1, 20] * width 的位置，而在的动画过程中不断 translateX -= 20，当滑过一个屏幕则重新初始化位置。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5df375b1e9af4814ac80ba506ff770b5~tplv-k3u1fbpfcp-zoom-1.image)

到这，梦幻的晴晚和流星效果就实现了，预告一下下期-炫酷的雷暴效果。

