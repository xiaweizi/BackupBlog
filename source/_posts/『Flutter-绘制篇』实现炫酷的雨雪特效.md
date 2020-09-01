---
title: 『Flutter-绘制篇』实现炫酷的雨雪特效
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: '1840'
date: 2020-09-01 22:11:12
---

### 前言

前不久，利用周末时间学习并完成一个简单的 Flutter 项目 -  [**简悦天气**](https://juejin.im/post/6866400501290926094)，*简约不简单，丰富不复杂*，这是一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径以及生活指数等服务，支持定位、删除、搜索等操作。

<!-- more -->

下图为主页效果：

![图1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76bdbd8781d64fe49f253207020b53b4~tplv-k3u1fbpfcp-zoom-1.image)

### 开始

项目中很多自定义 widget，今天的主角是 **背景层**，不同的天气气象有不一样的呈现效果，一共实现了 **12** 种类别，其中有 晴、多云、阴天、小中大雨、小中大雪、雾、霾、浮尘，而背景层又分为三层：

- 背景颜色层。从上到下的渐变效果
- 云层。只有一种图片，对其位移、数量、染色做不同变化达到不同效果
- 雨雪层。为雨雪天气单独做了动画，很炫酷。

好，真正的主角就是这个**雨雪层**，为了更好的预览效果，在关于页面有上角添加切换天气类型的入口，实时查看不同气象下不同的背景效果。如下图，为雨雪的最终效果(gif 效果看起来会失真，请下载 [apk](http://xiaweizi.top/SimplicityWeather-1_7.apk) 自行体验)：

![图5](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37cf495377b146cc9212cc49853c5591~tplv-k3u1fbpfcp-zoom-1.image)

![图6](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/478eccd10f79400690ca115b8c764be5~tplv-k3u1fbpfcp-zoom-1.image)

不得不说，如此复杂的动画(复杂并不是指多难实现，而是**不停**的绘制**很多**图片下)，Flutter 还能有不错的性能表现，媲美原生效果。

### 效果实现

这里不赘述绘制和动画相关知识，网上已经有很多文章介绍，本篇只针对项目中用到的实现方式和相关知识进行讲述，具有一定的局限性，适合简单的绘制动画逻辑。

#### 创建绘制类

因为 Flutter 处处是 widget，自定义 View 需要用到的是 CustomPaint，而成员变量中需要传入实现 CustomPainter 的类，那咱们先创建此类。

```dart
class RainSnowPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return true;
  }
}
```

有没有很熟悉，看到了 Canvas 的类，自然也有 Paint 类，有了画笔和画板，剩下就好办了。

#### 构造雨雪对象

对需要实现的效果进行分析，首先雨雪效果是由一张图片不同属性拼接而成，每个雨滴和雪花落实在屏幕上，必须有 x,y 的坐标属性。为了营造**远近**的效果，需要加上 scale 值，由于更加还原真实的视觉效果，雨滴的远近，必然**速度**上和**清晰度**上会有差异，因此加上 speed 和 alpha 属性，再加上其他计算用的属性，最后类的声明如下：

```dart
class RainSnowParams {
  double x;
  double y;
  double speed;
  double scale;
  double width;
  double height;
  double alpha;
  WeatherType weatherType;

  RainSnowParams(this.width, this.height, this.weatherType);
}
```

#### 属性初始化

有了属性后，接下来就是对属性进行赋值，为了保证效果更加的还原，所有属性既要有规则，又要随机。怎么解释规则性和随机性都要同时拥有，就拿雨速而言，小雨相对于大雨，雨的速度稍慢，但是不能很慢，并且每滴雨滴的速度不一样，这就致使小雨的速度必然在一个区间下随机，同样雪也一样。

初始化又分成两步，第一次的初始化和雨滴下落结束后的数据重置，实际上两者的区别只在于 y。第一次初始化 y 在屏幕高度中随机放置，而雨滴下落结束后，y 值置为0。那么就可以把重置逻辑封装统一的方法。

```dart
void reset() {
  double initScale = 0.1;
  double gapScale = 0.2;
  double initSpeed = 40;
  double gapSpeed = 40;
  if (weatherType == WeatherType.lightRainy) {
    initScale = 1.05;
    gapScale = 0.1;
    initSpeed = 15;
    gapSpeed = 10;
  } else if(){
    ...// 其他雨雪情况
  }
  double random = Random().nextDouble();
  this.scale = initScale + gapScale * random;
  this.speed = initSpeed + gapSpeed * (1 - random);
  this.alpha = 0.1 + 0.9 * random;
  x = Random().nextInt(width * 1.2 ~/ scale).toDouble() - width * 0.1 ~/ scale;
}
```

其中 init 代表这**初始值**，gap 代表**浮动值**，这两个根据雨雪量大小而做不同区分。通过 `Random().nextDouble() `获取随机 `[0.0, 1.0]` 的值，`random * gap + init` 就是最终的值。

x 的属性控制在 `[-0.1*width, 1.1 width] `的区间内随机，y 值上面已提到。

雪相对有雨有个不同，雨是垂直下落，而雪是随风摇摆，那为了营造这种感觉，此时就需要借助 `sin` 函数。

```dart
if (WeatherUtil.isSnow(_state.widget.weatherType)) {
    double offsetX = sin(params.y / (300 + 50 * params.alpha)) * (1 + 0.5 * params.alpha);
    params.x += offsetX;
}
```

#### 开始绘制

终于到了最重要的步骤 **绘制**，但他并不是最难的，有了前面创建好的**属性**和对其**初始化**，剩下就只是调用 api 进行绘制即可。不过再此之前好像漏了什么没说，没错，就是 **动画**，一个无限循环的动画。

Flutter 中创建动画也很简单，需要在动画监听中，判断如果动画结束则重新继续执行即可。 

```dart
1. 在 initState 函数中初始化 controller， animation 和 listener
    _controller =
        AnimationController(duration: Duration(minutes: 1), vsync: this);
    CurvedAnimation(parent: _controller, curve: Curves.linear);
    _controller.addListener(() {
      setState(() {});
    });
    _controller.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        _controller.repeat();
      }
    });
    _controller.forward();
2. 在 dispose 函数中释放掉动画资源
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
```

在初始化是便让他执行并一直执行知道页面销毁，有了动画后，开始进行绘制，雨雪的绘制逻辑基本相似，只不过图片源不一样。

别看屏幕上有很多雨滴，其实只用了一直图片，通过控制 alpha、speed和scale 的属性来随机展现不同的形态。还有，根据气象大中小雨类型的区分，会直接落实到雨滴数量和雨滴形态上的变化，营造出多样的差异。

```dart
  void drawRain(Canvas canvas, Size size) {
    weatherPrint(
        "开始绘制雨层 image:${_state._images?.length}, rains:${_state._rainSnows?.length}");
    if (_state._images != null && _state._images.length > 1) {
      ui.Image image = _state._images[0];
      if (_state._rainSnows != null && _state._rainSnows.isNotEmpty) {
        _state._rainSnows.forEach((element) {
          move(element);
          ui.Offset offset = ui.Offset(element.x, element.y);
          canvas.save();
          canvas.scale(element.scale, element.scale);
          var identity = ColorFilter.matrix(<double>[
            1, 0, 0, 0, 0,
            0, 1, 0, 0, 0,
            0, 0, 1, 0, 0,
            0, 0, 0, element.alpha, 0,
          ]);
          _paint.colorFilter = identity;
          canvas.drawImage(image, offset, _paint);
          canvas.restore();
        });
      }
    }
  }
```

这里绘制逻辑只用到了 drawImage 的方法，参数分别为图片、位置和画笔，不像 Android 提供了 paint.setAlpha() 的方法控制图片的透明值，这里需要通过 colorFilter 修改矩阵中对应的值来控制 alpha。

move() 函数用于控制雨滴在运动过程中 x和y 值的不断变化。

```dart
  void move(RainSnowParams params) {
    params.y = params.y + params.speed;
    if (WeatherUtil.isSnow(_state.widget.weatherType)) {
      double offsetX = sin(params.y / (300 + 50 * params.alpha)) * (1 + 0.5 * params.alpha);
      params.x += offsetX;
    }
    if (params.y > 800 / params.scale) {
      params.y = 0;
      if (WeatherUtil.isRainy(_state.widget.weatherType) &&
          _state._images.isNotEmpty &&
          _state._images[0] != null) {
        params.y = -_state._images[0].height.toDouble();
      }
      params.reset();
    }
  }
```

该方法每次重绘时都会调用，即 `y += speed` 根据 speed 不断修改 y 的属性，因为雪的特殊性，x 会通过 sin 函数运算后得出。以及当雨滴超过屏幕需要重新归位并重新初始化。

到此， 雨雪的绘制和动画逻辑已经讲述结束，是不是很简单，但是效果上还是相当酷炫的，感兴趣的可以到 [SimplicityWeather](https://github.com/xiaweizi/SimplicityWeather) 下载进行查看更多效果。最后再看看大雨下的效果。

![大雨特效](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6529e0543da34a188c8ce68438ceab3a~tplv-k3u1fbpfcp-zoom-1.image)