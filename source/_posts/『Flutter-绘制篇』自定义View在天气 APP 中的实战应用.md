---
title: 『Flutter-绘制篇』自定义View在天气 APP 中的实战应用
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: 26ab
date: 2020-09-05 16:11:12
---


## 前言

前不久，利用周末时间学习并完成一个简单的 Flutter 项目 -  [**简悦天气**](https://juejin.im/post/6866400501290926094)，*简约不简单，丰富不复杂*，这是一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径以及生活指数等服务，支持定位、删除、搜索等操作。

<!-- more -->

下图为主页效果，可以 [点击这里](http://xiaweizi.top/SimplicityWeather-2_1.apk) 进行下载 apk 体验：

![图1](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/76bdbd8781d64fe49f253207020b53b4~tplv-k3u1fbpfcp-zoom-1.image)

## 开始

本身作为天气 APP，**自定义绘制**自然少不了，首页多样的背景效果，[炫酷的雨雪效果](https://juejin.im/post/6867489001809379335)，展示当前空气质量和体感的圆环效果，动态**温度折线图**和日出日落图。

其实 [pub.dev](https://pub.flutter-io.cn/packages?q=chart) 上已经有不少 chart 插件，提供丰富的图表类型，支持各种动画和手势。但是如果是像本项目，使用场景并不需要手势，且没有复杂的动画，只存在折线这种形态，完全可以自己实现。一方面可以巩固和拓展 flutter 的绘制相关知识点，另一方面根据自己的实际需求，可以拥有更多的定制化功能。

先看一下最终效果，其中包括：

- 动态降雨折线图
![rain_chart](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e38f0489246496db52ce31e9e3e3abd~tplv-k3u1fbpfcp-zoom-1.image)
- 多日折线图
![day_chart](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abc8c5f578794682b3f5a07953e7f006~tplv-k3u1fbpfcp-zoom-1.image)
- 24小时折线图
![hour_chart](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51a97b4b4fcd42c2b88547cdf82746a1~tplv-k3u1fbpfcp-zoom-1.image)
- AQI圆弧
![aqi_chart](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/361b5ca437dc475f8f563867cdbf124f~tplv-k3u1fbpfcp-zoom-1.image)
- 日出日落图
![sun_chart](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4605e70d1ca54fa4b9470c4a20397c62~tplv-k3u1fbpfcp-zoom-1.image)



## 绘制

接下来，会以上述效果作为切入点，由简到难，由静态到动态，逐步分析绘制前数据的准备和绘制时相关接口调用，最后，总结出折线图绘制的**通用思路**，对后续有相关需求的小伙伴提供帮助。

### AQI圆弧

![aqi_chart](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/361b5ca437dc475f8f563867cdbf124f~tplv-k3u1fbpfcp-zoom-1.image)

先从最简单圆弧图开始，如上图可看到的信息有：半透明的圆弧，纯白色的圆弧，居中的 AQI 值以及其底部的文字描述。对于此图而言，只需要知道 ratio: 白色圆弧占比、AQIValue 和 AQIDesc。

这个简单直接先上代码再分析。

```dart
  @override
  void paint(Canvas canvas, Size size) {
    weatherPrint("AqiChartPainter size:$size");
    var radius = size.height / 2 - 10;
    var centerX = size.width / 2;
    var centerY = size.height / 2;
    var centerOffset = Offset(centerX, centerY);
    // 绘制半透明圆弧
    _path.reset();
    _path.addArc(Rect.fromCircle(center: centerOffset, radius: radius),
        pi * 0.7, pi * 1.6);
    _paint.style = PaintingStyle.stroke;
    _paint.strokeWidth = 4;
    _paint.strokeCap = StrokeCap.round;
    _paint.color = Colors.white38;
    canvas.drawPath(_path, _paint);
    // 绘制纯白色圆弧
    _path.reset();
    _path.addArc(Rect.fromCircle(center: centerOffset, radius: radius),
        pi * 0.7, pi * 1.6 * ratio);
    _paint.color = Colors.white;
    canvas.drawPath(_path, _paint);
    // 绘制 AQIValue
    var valuePara = UiUtils.getParagraph(value, 30);
    canvas.drawParagraph(
        valuePara,
        Offset(centerOffset.dx - valuePara.width / 2,
            centerOffset.dy - valuePara.height / 2));
    // 绘制 AQIDesc
    var descPara = UiUtils.getParagraph("$desc", 15);
    canvas.drawParagraph(
        descPara,
        Offset(centerOffset.dx - valuePara.width / 2,
            centerOffset.dy + valuePara.height / 2));
  }
```

将步骤进行分解：

1. 先绘制**半透明圆弧**，确认**中心点坐标**和半径，通过 `_path.addArc(Rect oval, double startAngle, double sweepAngle)` 方法进行绘制。oval: 圆弧所在矩形，startAngle: 起始角度(以钟表为例，0为3点方向)，sweepAngle: 划过角度(默认方向顺时针)。

2. 在半透明圆弧基础上，根据 ratio (`currentAqiValue / totalAqiValue`) 绘制**纯白色圆弧**。

3. 依次绘制中间 **AQIValue** 和 **AQIDesc**。Flutter 绘制文本跟 Android 比起来略微有点麻烦，通过构造 `ui.Paragraph` 对象，然后调用 `canvas.drawParagraph(Paragraph paragraph, Offset offset)` 方法进行绘制。一般通过封装好的静态初始化方法构建 `ui.Paragraph` 对象：

   ```dart
     static ui.Paragraph getParagraph(String text, double textSize,
         {Color color = Colors.white, double itemWidth = 100}) {
       var pb = ui.ParagraphBuilder(ui.ParagraphStyle(
         textAlign: TextAlign.center, //居中
         fontSize: textSize, //大小
       ));
       pb.addText(text);
       pb.pushStyle(ui.TextStyle(color: color));
       var paragraph = pb.build()..layout(ui.ParagraphConstraints(width: itemWidth));
       return paragraph;
     }
   ```

**关键词：** `addArc`、`Paragraph`、 `drawParagraph`

### 日出日落贝塞尔曲线

![sun_chart](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4605e70d1ca54fa4b9470c4a20397c62~tplv-k3u1fbpfcp-zoom-1.image)

上图看起来像是圆弧，其实是使用二阶贝塞尔曲线进行绘制。图中涵盖的信息并不多，其中包括左右日出日落时间、整体虚曲线、动态实曲线和当前时间。对于需要的数据除了日出日落时间，还需要根据 `(nowTime - sunriseTime)/(sunsetTime - sunriseTime)` 获取占比 ratio。

继续分解步骤：

1. 绘制 **虚曲线**，首先确认起点和终点，通过 `_path.quadraticBezierTo(double x1, double y1, double x2, double y2)` 绘制贝塞尔曲线，参数需要传入 **控制点** 坐标和 **终点** 坐标。很遗憾 Flutter 没有提供虚线的接口，借用 `path_drawing` 插件中的 `dashPath(Path source, {@required CircularIntervalList<double> dashArray,DashOffset dashOffset,})` 方法进行虚线的绘制。

   ```dart
   var height = size.height;
   var width = size.width;
   double startX = marginLeftRight;
   double startY = height - marginBottom;
   double endX = width - marginLeftRight;
   double endY = startY;
   _path.reset();
   _path.moveTo(startX, startY);
   _path.quadraticBezierTo(width / 2, marginTop, endX, endY);
   _paint.color = Colors.white;
   _paint.style = PaintingStyle.stroke;
   _paint.strokeWidth = 1.5;
   canvas.drawPath(
     dashPath(_path, dashArray: CircularIntervalList<double>([10, 5])),
     _paint);
   ```

2. 绘制 **实虚线**，这里遇到一个问题，已知比例 ratio，在虚曲线上绘制实曲线(保证重叠)，不同于直线或者弧线，通过控制 xy 或者 sweepAngle 轻松实现。对二阶贝塞尔曲线稍有了解的可以知道，其主要由起始点和控制点组成，这三个值稍有变化，都很难做到重叠，所以得另辟蹊径。

   Android 中有 PathMeasure 可以对 Path 进行分段，然后根据需要绘制的段数进行控制。同样，Flutter 也有对应的 API：

   ```dart
   var metrics = _path.computeMetrics();
   var pm = metrics.elementAt(0);
   Offset sunOffset = pm.getTangentForOffset(pm.length * ratio).position;
   canvas.save();
   canvas.clipRect(Rect.fromLTWH(0, 0, sunOffset.dx, height));
   canvas.drawPath(_path, _paint);
   canvas.restore();
   ```

   通过 getTangentForOffset 得到 ratio 下在曲线上的 x,y 坐标点，然后 `_path.clipRect()` 对虚曲线裁剪最终得到实曲线。

3. 绘制**小太阳和当前时间**，知道曲线上的 x,y 坐标，这就好办了

   ```dart
   _paint.style = PaintingStyle.fill;
   _paint.color = Colors.yellow;
   canvas.drawCircle(sunOffset, 6, _paint);
   
   var now = DateTime.now();
   String nowTimeStr = "${now.hour}:${now.minute}";
   var nowTimePara = UiUtils.getParagraph(nowTimeStr, 14);
   canvas.drawParagraph(nowTimePara,
                        Offset(sunOffset.dx - nowTimePara.width / 2, sunOffset.dy + 10));
   ```

   **关键词：** `quadraticBezierTo`、`dashPath`、`computeMetrics`、`getTangentForOffset`、`clipRect`、`drawCircle`

### 多日折线图

- 多日折线图
![day_chart](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abc8c5f578794682b3f5a07953e7f006~tplv-k3u1fbpfcp-zoom-1.image)

上下的文字区域绘制根据各自高度顺延绘制即可，只要预留出中间折线的绘制区域即可。中间的折线区域又可以继续平分成 top 和 bottom 两个折线，各自绘制各自的，互不干扰。

折线图的绘制思路分为三步：找出最大最小值、计算单位温度的 y 值和遍历绘制

1. 遍历找出 top 和 bottom 的**最大最小值**

   ```dart
   void setMinMax() {
     _data.forEach((element) {
       if (element.dayTemp > topMaxTemp) {
         topMaxTemp = element.dayTemp;
       }
       if (element.dayTemp < topMinTemp) {
         topMinTemp = element.dayTemp;
       }
       if (element.nightTemp > bottomMaxTemp) {
         bottomMaxTemp = element.nightTemp;
       }
       if (element.nightTemp < bottomMinTemp) {
         bottomMinTemp = element.nightTemp;
       }
     });
   }
   ```

2. **根据温度计算x,y值**，目前已知折线的高度 itemHeight, 具体温度 temp,起点 topLineStartY，最高最低温度已经实际温度，即可算出温度对应的 y 坐标值，x坐标值

   ```dart
   getTopLineY(int temp) {
     if (temp == topMaxTemp) {
       return topLineStartY;
     }
     return topLineStartY +
       (topMaxTemp - temp) / (topMaxTemp - topMinTemp) * lineHeight;
   }
   x = startX + index*itemWidth;
   ```

3. **开始绘制**，x,y 都知道了，直线、原点以及文字都可以进行遍历绘制了

   ```dart
   _paint.color = Colors.white;
   var topOffset = Offset(startX, getTopLineY(element.dayTemp));
   var bottomOffset = Offset(startX, getBottomLineY(element.dayTemp));
   _paint.style = PaintingStyle.fill;
   // 绘制折线上的圆点
   canvas.drawCircle(topOffset, 3, _paint);
   canvas.drawCircle(bottomOffset, 3, _paint);
   
   // 绘制圆点上下的温度值
   var topTempPara = UiUtils.getParagraph("${element.dayTemp}°", mainTextSize, itemWidth: itemWith);
   canvas.drawParagraph(
     topTempPara, Offset(topOffset.dx - topTempPara.width / 2, topOffset.dy - topTempPara.height - 5));
   var bottomTempPara = UiUtils.getParagraph("${element.dayTemp}°", mainTextSize, itemWidth: itemWith);
   canvas.drawParagraph(
     bottomTempPara, Offset(bottomOffset.dx - bottomTempPara.width / 2, bottomOffset.dy + 5));
   
   // 绘制折线
   if (index == 0) {
     _topPath.moveTo(topOffset.dx, topOffset.dy);
     _bottomPath.moveTo(bottomOffset.dx, bottomOffset.dy);
   } else {
     _topPath.lineTo(topOffset.dx, topOffset.dy);
     _bottomPath.lineTo(bottomOffset.dx, bottomOffset.dy);
   }
   startX += itemWith;
   });
   _paint.strokeWidth = 2;
   _paint.style = PaintingStyle.stroke;
   canvas.drawPath(_topPath, _paint);
   canvas.drawPath(_bottomPath, _paint);
   }
   ```

**关键词：** 最大最小值

### 动态降雨折线图

![rain_chart](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e38f0489246496db52ce31e9e3e3abd~tplv-k3u1fbpfcp-zoom-1.image)

终于到了今天最难的角登场，只是对比前几个比较难，在上述折线的基础上加了折线入场动画。话不多说咱们开始吧，上图可拆成三部分，背景(y轴，xy轴描述)、渐变折线和动画

#### 背景

x 轴被二等分，y 轴被三等分，计算出 xItemWidth 和 yItemHeight，然后绘制线和文字

```dart
void drawBg(Canvas canvas, Size size) {
  // 绘制背景 line
  double itemHeight = (size.height - _marginBottom) / 3;
  double bgLineWidth = size.width - _marginLeft - _marginRight;
  _paint.style = PaintingStyle.stroke;
  _paint.strokeWidth = 1;
  _paint.color = Colors.white.withAlpha(100);
  for (int i = 0; i < 4; i++) {
    var startOffset = Offset(_marginLeft, itemHeight * i);
    var endOffset = Offset(_marginLeft + bgLineWidth, itemHeight * i);
    canvas.drawLine(startOffset, endOffset, _paint);
  }

  // 绘制底部文字
  var hourY = size.height - _marginBottom + _timeMarginTop;
  var nowPara = UiUtils.getParagraph("现在", _textSize, itemWidth: bgLineWidth / 3);
  canvas.drawParagraph(nowPara, Offset(_marginLeft - nowPara.width / 2, hourY));
  var onePara = UiUtils.getParagraph("1小时后", _textSize, itemWidth: bgLineWidth / 3);
  canvas.drawParagraph(onePara, Offset(_marginLeft + bgLineWidth / 2 - onePara.width / 2, hourY));
  var twoPara = UiUtils.getParagraph("2小时后", _textSize, itemWidth: bgLineWidth / 3);
  canvas.drawParagraph(twoPara, Offset(_marginLeft + bgLineWidth - twoPara.width / 2, hourY));

  // 绘制左侧文字
  var bigPara = UiUtils.getParagraph("大", _textSize);
  canvas.drawParagraph(bigPara, Offset(_marginLeft / 2 - bigPara.width / 2, 0));
  var middlePara = UiUtils.getParagraph("中", _textSize);
  canvas.drawParagraph(middlePara, Offset(_marginLeft / 2 - middlePara.width / 2, itemHeight));
  var smallPara = UiUtils.getParagraph("小", _textSize);
  canvas.drawParagraph(smallPara, Offset(_marginLeft / 2 - smallPara.width / 2, itemHeight * 2));

}
```

#### 渐变折线

1. **绘制折线**，最大值不用计算已经知道 yMax = 1.0，xMax = 120，可以计算出点的 x,y 坐标值，然后进行遍历绘制

   ```dart
   double width = size.width - _marginLeft - _marginRight;
   double height =  size.height - _marginBottom;
   double startX = _marginLeft;
   double itemWidth = width / 120;
   double itemHeight = height / 100;
   _linePath.reset();
   for (int i = 0; i < _data.length; i++) {
     double y = height - _data[i] * 100 * itemHeight * _ratio;
     double x = startX + i * itemWidth;
     if (i == 0) {
       _linePath.moveTo(x, y);
     } else {
       _linePath.lineTo(x, y);
     }
   }
   _linePaint.style = PaintingStyle.stroke;
   _linePaint.strokeWidth = 1;
   _linePaint.color = Colors.white;
   canvas.drawPath(_linePath, _linePaint);
   _linePath.lineTo(width + startX, height);
   _linePath.lineTo(startX, height);
   _linePath.close();
   ```

2. **渐变效果**，复用折线 path，通过 `ui.Gradient.linear` 创建渐变区域，然后设置到 `_linePaint.shader` 上

   ```dart
   var gradient = ui.Gradient.linear(
     Offset(0, 0),
     Offset(0, height),
     <Color>[
       const Color(0xFFffffff),
       const Color(0x00FFFFFF)
     ],
   );
   _linePaint.style = PaintingStyle.fill;
   _linePaint.shader = gradient;
   canvas.drawPath(_linePath, _linePaint);
   ```

#### 入场动画

在 **渐变折线#1** 中对 y 的计算 `double y = height - _data[i] * 100 * itemHeight * _ratio;` 中提到了 _ratio，这个就是控制动画效果关键变量，区间 [0,1]，0为`y=0.0` 的直线，1为实际的折线图效果。

而这个 _ratio 有动画进行控制:

```dart
_controller =
  AnimationController(duration: Duration(milliseconds: 250), vsync: this);
CurvedAnimation(parent: _controller, curve: Curves.linear);
_controller.addListener(() {
  setState(() {
    _ratio = _controller.value;
  });
});
```

最终的动态折线效果即可完成。

**关键词：**`drawLine`、` ui.Gradient.linear`、`AnimationController`

## 总结

整体下来，无论是圆弧、曲线还是折线或者类似简单的绘制都有章可循。

1. 对 待实现效果进行分析，找出关键信息进行分层分步，找出静态数据和动态数据，也就是常量和变量。
2. 计算好基础数据，比如整体宽高，单位宽高，起始值，最大最小值
3. 有了数据支撑，根据效果调用对应的绘制 API，设置 paint 的相关属性，完成绘制
4. 如果有动画，以控制变量作为切入口，动画本身只关注变量值的改变，而不用考虑变量对绘制的影响