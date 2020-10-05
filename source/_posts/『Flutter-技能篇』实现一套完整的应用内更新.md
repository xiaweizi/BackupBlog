---
title: 『Flutter-技能篇』实现一套完整的应用内更新
author: 下位子
tags:
  - Flutter
  - 跨平台
categories:
  - Flutter
abbrlink: '8200'
date: 2020-10-05 19:11:12
---

## 前言

前不久，利用周末时间学习并完成一个简单的 Flutter 项目 -  [**简悦天气**](https://github.com/xiaweizi/SimplicityWeather)，*简约不简单，丰富不复杂*，这是一款简约风格的 flutter 天气项目，提供实时、多日、24 小时、台风路径、语音播报以及生活指数等服务，支持定位、删除、搜索等操作。

<!-- more -->

下图为主页效果，[点击下载](http://xiaweizi.top/SimplicityWeather-2_6.apk) 进行体验：

![home](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6471e071336e42e38f9037c3b5e5ae45~tplv-k3u1fbpfcp-zoom-1.image)

一款成熟的 APP，为了保证用户手上的 `apk` 始终是最新版本，一方面可以通过发布到各产商应用商店，依赖其自升级能力完成自更新；或者，自己实现一套应用内更新逻辑，两者各有利弊。

前者，优势在于有厂商应用商店自升级通道，可以静默安装，用户基本无感知，但是，如果用户关闭自更新并且不主动更新，那么 app 永远没有自升级的可能性，而且每家都发布维护，成本也是相当高的。

而后者自己实现，虽然不能静默安装，但是可以在用户每次打开 `app` 时，根据业务需求或者版本更新，主动推送新版本，提醒用户相关问题的修复，或者有新的功能，对提升 `app` 的粘性有很大的帮助。

我们今天将全面完整的实现一套 `Flutter` 应用内更新流程，涵盖了**前端、服务端和客户端**部分，所以本篇文章主要有两个主题：

1. 实现一套**完整**的应用内更新流程
2. 实现**炫酷**的更新弹窗动画

希望能帮助到有需要的小伙伴。

## 开始

咱们直接先看一下客户端完成后的效果：

![app_update](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f772d4efc8d4879ac80941579196d1d~tplv-k3u1fbpfcp-zoom-1.image)

如上图所示，在 `app` 打开时，会提升有新版本更新，并告知最新版本的更新内容。

点击立即更新，会获取配置的下载地址进行下载，并根据当前下载进度呈现水波纹的效果，在加上炫酷的背景动画，整体给人非常炫酷的效果。(水波纹和背景动画后面也会进行讲解)

更新完成后，直接会跳转到应用的覆盖安装页面，点击安装则可以更新到最新版本啦。

开始正文，接下来将从 `apk` 打包，**前端**页面编写，**后端**接口返回以及**客户端**具体逻辑实现，并附带炫酷的更新弹窗效果，完成的呈现一整套流程的运转过程。

### APK 打包

第一步，虽然简单，但是必不可少，那就是生成最新的 `apk` 包，不过有不少细节点需要关注。

1. 在终端使用 `flutter build apk --target-platform android-arm64 --split-per-abi` 进行打包，任务执行完成后，会在 `build/app/outputs/apk/release/app-arm64-v8a-release.apk` 下生成 apk 文件。
	
    这里根据需要编译平台，我这边只需要 `arm64` 下的 `apk`，所以中添加 `android-arm64` 配置。

2. 记得更新 `pubspec.yaml` 中的 `version` 字段。 `version: 2.6.0+26` 2.6.0 代表版本名称，26 代表版本号，用于判断是否需要提醒更新。

3. 整理距离上一个版本的 change，看一下主要有哪些更新点，为后面更新说明做准备。

### 前端页面配置

之前自学过一点 `python`，就决定使用 [Django](https://www.djangoproject.com/) 来完成。

其实，自己一直有一个 `django` 的小项目，作为自己平常工具、或者爬虫后数据展示的平台。前端时间，媳妇怀孕期间，为了让我和媳妇能了解到离预产期的时间、宝宝状态和妈妈状态，怕去了妈妈帮的数据，每天定时定点推送当天的具体数据。不仅如此，娃生下来后，每天也会不间断的推送娃的出生天数和注意事项，确实还挺实用的。给大家简单的看一下后台数据和推送内容~

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c6bbeba5695e45529f085d0745705406~tplv-k3u1fbpfcp-zoom-1.image)

这是后台的数据，可以查看和配置各种信息。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f66f1b67cb3401bb0edbc198cfa4a63~tplv-k3u1fbpfcp-zoom-1.image)

这是通过邮箱推送后内容展示。

虽然作为 `Android` 开发，但还是要对前后端的知识点有大概了解，这样跟其他的小伙伴沟通起来障碍才会小一点。

稍微有点扯远了，本篇不介绍 `Django` 的使用和项目创建，有需要的可以私聊我要课程。

#### 创建表

自升级的表数据很简单，只需要 下载地址、版本号和更新点即可。

```python
class OTAData(models.Model):
    ota_url = models.CharField("下载地址", max_length=256)
    ota_app_code = models.CharField("版本号", max_length=100)
    ota_app_desc = models.TextField("更新点", default="")

    class Meta:
        db_table = 'otadata'
        verbose_name = 'OTA数据'
        verbose_name_plural = verbose_name
```

#### 创建后台展示

在每个模块下的 `admin.py` 中配置需要展示的字段，以及排序规则等。

```python
@admin.register(OTAData)
class OTAAdmin(ImportExportActionModelAdmin):
    resource_class = OTAResource
    list_display = ("ota_url", "ota_app_code", "ota_app_desc")
```

#### 配置数据

部署到线上后，输入绑定的域名或者 IP 地址访问对应的后台 url，在页面中配置相应的数据。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d39939eb9cba441c95efbf6cfac2643d~tplv-k3u1fbpfcp-zoom-1.image)

### 服务端数据下发

后端页面配置完成，此时需要服务端提供接口给客户端请求。[Django](https://www.djangoproject.com/) 同样提供能力支持。

在模块下的 `views.py` 中配置下发的内容：

```python
def ota_data(request):
    ota_data = OTAData.objects.all()
    inner_data = {}
    if ota_data.__len__() > 0:
        item = ota_data[0]
        inner_data = {
            "url": item.ota_url,
            "appCode": item.ota_app_code,
            "desc": item.ota_app_desc,
        }
    data = {
        "status": 0,
        "code": 200,
        "data": inner_data,
    }
    return JsonResponse(data, json_dumps_params={'ensure_ascii': False})
```

通过返回 `JsonResponse` 对象来控制返回的数据格式，对数据库中的数据进行组装后返回即可。

然后在模块下的 `urls.py` 配置地址访问格式:

```python
path('ota/', ota_data)
```

发布后，通过访问 `http://xxx/ota/` 既可请求到正确的数据。

```json
{
    "status": 0,
    "code": 200,
    "data": {
        "url": "http://xiaweizi.top/SimplicityWeather-2_6.apk",
        "appCode": "26",
        "desc": "- 新增城市管理动画效果\r\n- 优化搜索结果页展示效果\r\n- 新增炫酷的 demo 入口效果\r\n- 优化背景动画效果"
    }
}
```

### 客户端

> 对于 iOS 就直接跳转到 `App Store`， 本篇讲述 `Android` 端的实现逻辑。

对于客户端的逻辑如下：

1. app 启动时请求 ota 接口
2. 如果当前版本小于最新版本，则弹窗提示，并展示配置的更新内容
3. 点击「立即更新」，根据配置的下载地址进行下载，并实时返回下载进度
4. 下载成功，跳转到更新页面，提醒用户覆盖安装

#### 请求 ota 接口

```dart
  static initOTA() async {
    var otaData = await WeatherApi().getOTA();
    if (otaData != null && otaData["data"] != null) {
      String url = otaData["data"]["url"];
      String desc = otaData["data"]["desc"];
      String versionName = "";
      int appCode = int.parse(otaData["data"]["appCode"]);
      var packageInfo = await PackageInfo.fromPlatform();
      var number = int.parse(packageInfo.buildNumber);
      if (appCode > number) {
        showOTADialog(url, desc, versionName);
      }
    }
  }
```

成功请求后，根据版本号字段进行判断，如果当前版本小于最新版本，则 `showOTADialog`。

#### 下载文件并更新进度

因为下载和安装涉及到文件的存储需要在 `AndroidManifest` 中声明相应的权限，和动态申请存储权限。

```xml
  <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
  <uses-per
```