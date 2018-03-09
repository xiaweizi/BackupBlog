---
title: hexo 博客小功能添加-评论、相册、字数统计...
date: '2018.02.27 20:41:25'
categories:
  - 技术分享
tags:
  - hexo
abbrlink: 31905
---


## 前言

前段时间 [个人博客全新上线(阿里域名+GitPages+hexo+Yelee)](https://www.jianshu.com/p/502765617b74) 搭建的博客 [我的博客](http://xiaweizi.cn),基础的功能已经实现了，想着既然有了自己的博客就要好好的维护(折腾)一下，于是便参考着别人的博客添加一些额外的小功能。

<!-- more -->

首先我的博客是基于[yelee](https://github.com/MOxFIVE/hexo-theme-yelee)风格，作者也是在 [Hexo-Theme-Yilia](https://github.com/litten/hexo-theme-yilia)的基础上进行修改，增加了一些新的功能，个人也是喜欢这种简洁的风格，觉得还不错就使用了该主题，在此表示对作者无私贡献的感谢！展示一下该主题风格的特性:

|  | New |
| - | :- | 
| 1 | 嵌入边栏的文章目录 |
| 2 | 透明化背景，随机背景大图 |
| 3 | 页内跳转按钮 |
| 4 | 文章版权等信息显示 |
| 5 | 文章导航切换按钮与文章迷你列表 |
| 6 | 网站计数 |
| 7 | 多语言支持 |
| 8 | 本地站内实时搜索 |
| 9 | 动态加载评论 |

具体的使用可以参考一下[官方使用介绍](http://moxfive.coding.me/yelee/)。

为了给博客添加一些有趣的功能，就四处查资料，借鉴别人的博客，陆陆续续添加了一些小的功能，在此介绍一下添加的过程，或许对于感兴趣的您有所帮助，后续会陆续添加...

目前添加了:

- 评论
- 相册
- 文章字数统计和阅读时长
- 网易云音乐
- 鼠标点击效果

## 来必力评论

### 介绍

[yelee](https://github.com/MOxFIVE/hexo-theme-yelee) 原生就已经支持了 `Disqus`、多说和友言，具体的使用可以参考[yelee-评论系统](http://moxfive.coding.me/yelee/2.Basic-Usage/comment.html),但是，鉴于多说已经关闭，友言支持又不太友好，于是一开始就集成了`Disqus`。不过后来发现手机上展示不出来，一研究原来还得翻墙，这显然不够友好，上网了查了查还有网易云跟帖和来必力两个评论系统，对比了一下，最终选择了[来必力](https://livere.com)。

![来必力首页](http://upload-images.jianshu.io/upload_images/4043475-650a5085c30ea485.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![数据分析](http://upload-images.jianshu.io/upload_images/4043475-16bff39e7d148364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![代码](http://upload-images.jianshu.io/upload_images/4043475-4b3cecd8e6b109bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


无论是功能还是数据分析还是UI都很不错。

### 集成

因为原先的主题没有集成来必力，所以需要自己去集成，作为一个 `Android` 程序员，对这方面一窍不通，所以只能按部就班，照着其他的评论系统处理。

#### 1. 主题站点添加 uid

打开`theme/yelee/_config.yml`，添加配置信息。

     livere:
        on: true
        livere_uid: Your uid

#### 2. 创建评论 ejs 文件

在`themes/yelee/layout/_partial/comments`文件夹创建`livere.ejs`文件，拷贝上图的代码。

```
<section class="livere" id="comments">
    <!-- 来必力City版安装代码 -->
    <div id="lv-container" data-id="city" data-uid="Your uid">
    <script type="text/javascript">
   (function(d, s) {
       var j, e = d.getElementsByTagName(s)[0];

       if (typeof LivereTower === 'function') { return; }

       j = d.createElement(s);
       j.src = 'https://cdn-city.livere.com/js/embed.dist.js';
       j.async = true;

       e.parentNode.insertBefore(j, e);
       })(document, 'script');
    </script>
    <noscript> 为正常使用来必力评论功能请激活JavaScript</noscript>
    </div>
    <!-- City版安装代码已完成 -->
</section>
```

#### 3. 配置文章内的评论部分内容

打开`themes/yelee/layout/_partial/article.ejs`。

![article](http://upload-images.jianshu.io/upload_images/4043475-77ff3ca9a7cf3e5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加红色框标记的代码，到这就可以进行评论了，效果如下:

![评论](http://upload-images.jianshu.io/upload_images/4043475-dcf7a8397fd1a0dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大家可以到我的[留言板](http://xiaweizi.cn/message/)进行留言🤣。

## 相册

### 介绍

如果你跟我一样，对前端一窍不通，但是还是想添加相册功能，就去看看别人已经实现的案例，然后进行模仿~😆。[yilia](https://github.com/litten/hexo-theme-yilia)的作者已经实现了，效果很不错，不妨可以学习一下，我也是照着资料借鉴了一下。

### 集成

#### 1. 创建可以点击链接

如果想单独创建一个可以单击的链接，方法是相同的，首先进入您的博客目录执行

```
hexo new page photos
```

立即在`source`下生成`photos/index.md`文件，然后在主题站点的配置文件`theme/yelee/_config.yml`中添加点击的文案和跳转到位置。

![相册](http://upload-images.jianshu.io/upload_images/4043475-9433d49dd33043b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2. 拷贝他人的 photos 内容

可以参考别人做好的博客，找到他的博客备份[博客备份](https://github.com/litten/BlogBackup),下载`source/photos`下的所有文件，拷贝到你对应的位置。

![拷贝配置](http://upload-images.jianshu.io/upload_images/4043475-6820c4cefbfd74aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一般只需要修改`ins.js`文件的`render()`函数。这个函数式用来渲染数据的，里面配置了展示图片的链接地址。

![图片链接地址](http://upload-images.jianshu.io/upload_images/4043475-5276a7b36a5116e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这个时候界面已经出来，接下来就是放置您的图片。

#### 3. 处理图片并引入图片

接下来就是最后一步，也是最重要的一步了，使用`python`写的脚本，生成一套大图和一套小图，然后上传到`Github`上，随即生成`json`文件，这个文件保存在`source/photos/data.json`。

脚本可以直接下载 [下载地址](https://github.com/maker997/backupBlog/)，整个文件目录：

![文件目录](http://upload-images.jianshu.io/upload_images/4043475-40f1d984fcf69d89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
if __name__ == "__main__":
    cut_photo()        # 裁剪图片，裁剪成正方形，去中间部分
    compress_photo()   # 压缩图片，并保存到mini_photos文件夹下
    git_operation()    # 提交到github仓库
    handle_photo()     # 将文件处理成json格式，存到博客仓库中
```

主要的执行的就是上面的代码，前提需要您的电脑装 `python`。

最后执行

```
hexo clean (清理之前的 HTML 等)
hexo g (生成 HTML 文件)
hexo s (看看效果)
```

查看一下效果。

![相册](http://upload-images.jianshu.io/upload_images/4043475-13e6ce4a9ca5af54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 字数统计和阅读时长

### 介绍

Next 是已经集成了这个功能，所以还是得需要咱们自己完成，首先看一下官网的使用帮助[hexo-wordcount](https://www.npmjs.com/package/hexo-wordcount).

### 集成

#### 1. 安装 hexo-wordcount

    npm i --save hexo-wordcount

先安装插件。

### 2. 文件配置

在`yelee/layout/_partial/post/word.ejs`下创建`word.ejs`文件：

```
<div style="margin-top:10px;">
    <span class="post-time">
      <span class="post-meta-item-icon">
        <i class="fa fa-keyboard-o"></i>
        <span class="post-meta-item-text">  字数统计: </span>
        <span class="post-count"><%= wordcount(post.content) %>字</span>
      </span>
    </span>

    <span class="post-time">
      &nbsp; | &nbsp;
      <span class="post-meta-item-icon">
        <i class="fa fa-hourglass-half"></i>
        <span class="post-meta-item-text">  阅读时长: </span>
        <span class="post-count"><%= min2read(post.content) %>分</span>
      </span>
    </span>
</div>
```
然后在 `themes/yelee/layout/_partial/article.ejs`中添加

![article 配置](http://upload-images.jianshu.io/upload_images/4043475-8f87eb04040f35ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`word_count` 是主题`_config.yml`中配置是否需要添加字数统计功能控制 flag，`no_word_count`即配置文章是否需要显示字数统计的功能。

大功告成~看一下效果：⤵️

![首页展示](http://upload-images.jianshu.io/upload_images/4043475-a382b16fb0e50417.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![内容展示](http://upload-images.jianshu.io/upload_images/4043475-cbdad1ce42edc83f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 网易云音乐

### 介绍

作为一名`Android`开发者，最少不了的两个软件，一个是`Android Studio`，还有一个就是`网易云音乐`，那博客也要添加这个功能。

### 集成

集成起来就很简单了，`MarkDown` 是支持 `h5` 代码的，所以打开[网易云](https://music.163.com),输入你想要的歌曲，点击对应歌曲的 生成外链播放器。

![外链](http://upload-images.jianshu.io/upload_images/4043475-3a5ba250204c5338.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前提是有版权哈，然后拷贝相应的代码即可。

![拷贝代码](http://upload-images.jianshu.io/upload_images/4043475-b820e3f312ff2724.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

效果请点击[我的相册](http://xiaweizi.cn/photos/)

## 鼠标点击效果

废话不多说，直接看如何集成。

#### 1. 拷贝需要的文件

进入到[我的备份](https://github.com/xiaweizi/BackupBlog)，拷贝文件。

![点击效果文件](http://upload-images.jianshu.io/upload_images/4043475-15b1a79a486bf7d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

拷贝`resources`下的所有文件到您对应的目录。

#### 2. 添加配置

打开`themes/yelee/layout/_partial/after-footer.ejs`文件，添加刚刚添加文件的配置。

![配置](http://upload-images.jianshu.io/upload_images/4043475-26b534dbe4438243.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看一下效果：

![鼠标点击效果.gif](http://upload-images.jianshu.io/upload_images/4043475-e04028a13a498132.gif?imageMogr2/auto-orient/strip)


如果遇到什么问题，希望我可以帮助到你，[我的博客](http://xiaweizi.cn)