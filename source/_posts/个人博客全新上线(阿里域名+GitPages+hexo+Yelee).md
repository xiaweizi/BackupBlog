title: 个人博客全新上线(阿里域名+GitPages+hexo+Yelee)
date: 2018.02.14 20:46:25
categories:
- 技术分享
tags:
- 博客
- hexo
---

## 前言

新年新气象，狗年也是我本命年，因此决定将博客重新搞一波。其实之前已经搭好过博客，因为前段时间换了`mac`，之前的环境和博客的内容都没有备份，索性就重新开始吧。那展示一下我的新的博客地址:

[http://xiaweizi.cn](http://xiaweizi.cn)

<!-- more -->

在来一波动态图演示：

电脑端：

![电脑端.gif](http://upload-images.jianshu.io/upload_images/4043475-c2da9789c4e8a0c9.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



手机端：

![手机博客](http://upload-images.jianshu.io/upload_images/4043475-f2f6b7f7a5ac6a38.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


整个的搭建，一些的配置和博客的迁移也是花了我一整个下午和晚上加班的时间，自我感觉看起来很舒服，那接下来我就大概讲述一下这个博客搭建的过程。

## GitHub Pages + Hexo

没钱买域名，没时间网站备案，那就用`GitHub Pages`和`Hexo`搭建属于自己的博客吧。其实网上这个教程真的是一搜一大把，我之前也写过，不过被我弄丢了，所以我就不赘述了，贴几个链接以供参考，只要照着流程走，是肯定可以顺利搭建的，这是件一劳永逸的事情。

[1. 如何搭建一个独立博客](https://www.jianshu.com/p/05289a4bc8b2)
[2. 搭建属于自己的博客](https://www.jianshu.com/p/465830080ea9)

## 域名

域名，之前的云服务器套餐就是在[阿里云](https://www.aliyun.com/?spm=5176.8006371.388261.1.66867e63FewLSC)上购买，因此我就顺便在[阿里云](https://wanwang.aliyun.com/?spm=5176.8142029.735711.56.a72376f4MMmf6X)进行购买域名。

前缀的选择就没有想那么多，也没有想的那么商业化，直接就是本人姓名的中文拼音。至于后缀，一开始对这块不熟悉，就先购买一个最便宜的尝试一下`.top`，成功后，索性花点钱买了个`.cn`——全球唯一由中国管理的英文国际顶级域名。

![域名选择](http://upload-images.jianshu.io/upload_images/4043475-3ebf50b50d917461.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先选择你喜欢和合适的域名，然后进行购买，购买成功并通过实名审核后，就可以在控制台找到您刚刚注册的域名。


![解析](http://upload-images.jianshu.io/upload_images/4043475-e82acbfa03abcba6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我购买了两个，因此`xiaweizi.cn`和`xiaweizi.top`就会展示在首页，如果想绑定到`GitHub Pages`上,点击所选域名对应的「解析」按钮。

![添加解析](http://upload-images.jianshu.io/upload_images/4043475-849b16522051456a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里我之前已经添加过两个解析，第一次的时候需要你自行添加相应的解析。

![解析1](http://upload-images.jianshu.io/upload_images/4043475-6a6af64c6a80ca51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![解析2](http://upload-images.jianshu.io/upload_images/4043475-45ed7b185921b081.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照上图添加两条解析。方式有很多种，我选择**A**的记录类型，即将域名指向一个 IPV4 地址，那记录值对应的就是你的`GitHub Pages`对应的地址，那这个地址如何获取呢？

![获取 ip 地址](http://upload-images.jianshu.io/upload_images/4043475-756630c8d958e0c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直接通过`ping + "your name".github.io`的方式获取`IP`地址，也就是解析的记录值。

域名的配置已经完成了，剩下的就是`GitHub Pages`上的配置，有两种方式：

第一种，登录你的`GitHub`，找到名为`"your name".github.io`的仓库，点击`settings`的选项，找到`GitHub Pages`区域，即可看到`Custom Domian`的配置，这里输入您购买的域名即可。

![github pages](http://upload-images.jianshu.io/upload_images/4043475-9ad476d51ad48a3c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二种，在`hexo`的对应的目录下创建`CNAME`文件，里面输入您购买的域名。

![添加 cname 文件](http://upload-images.jianshu.io/upload_images/4043475-25916cb3e8311844.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不到一天的时间，即可通过审核并解析成功，直接就可以访问您的域名并定向到您的博客位置。

[xiaweizi](http://xiaweizi.cn)

到此，博客的大概框架和域名已经配置成功了，可以大致看到效果了。

## 主题选择

至于主题的选择完全是看个人的喜好了，有的人喜欢简洁大方，有的喜欢色彩张扬，有的喜欢品质内涵，找到自己合适的就行，如果实在没有满足您的胃口的，不妨可以根据自己的想法编写属于您自己的主题。

截止今日[hexo 主题](https://hexo.io/themes/)已经有 189 个样式供选择，或者可以参考知乎上的一篇文章选择：[有哪些好看的 hexo 主题](https://www.zhihu.com/question/24422335)。

我 选择了[yelee](https://github.com/MOxFIVE/hexo-theme-yelee)主题 —— 简而不减 Hexo 双栏博客主题，独具匠心的设计风格，突出内容为主题，凸显博主个性！

那接下来我就大概总结一下`yelee`主题的相关配置，当然官网是有详细的配置文档的：[中文使用文档](http://moxfive.coding.me/yelee/)，举一反三，其他的主题配置大同小异。

**使用 yelee 主题**

打开终端，`cd`到您`hexo/blog`的目录,

    git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee

下载`yelee`主题，那此时，分别在`hexo/blog`和`hexo/blog/theme/yelee`下有两个`_config.yml`的配置文件，前者是「站点配置文件」，后者是「主题配置文件」，大致所有的配置都在这里完成。

第一步就要在站点配置文件中，选择你刚下载的`yelee`主题。

![主题选择](http://upload-images.jianshu.io/upload_images/4043475-00bd80cf773d6bc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在`hexo/blog`目录下输入相应的`hexo`命令完成生成、测试和部署的工作。

    // 1. 生成
    hexo g
    // 2. 测试(localhost:4000)
    hexo s
    // 3. 部署
    hexo d

**编写文章**

![文章编写](http://upload-images.jianshu.io/upload_images/4043475-4f3148e75bdf86bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

头部即为每次编写文章的配置信息。

    title: 文章的标题
    date: 文章的创建时间
    updated: 文章的更新时间
    categories: 文章所属的分类(一般有且只有一个)
    tag: 文章所打的 tag(这个根据文章特点添加)

需要注意的是，主页展示每个文章，并不是展示摘要而是直接全文，这样的体验很不好，这个时候需要添加一行摘要分割标识：`<!-- more -->`

![摘要分割](http://upload-images.jianshu.io/upload_images/4043475-416e405032bfd6c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`<!-- more -->`之前的正文部分就是展示在主页的摘要。

**第三方服务**

首先在主题配置文件中找到`github_widget:`并设置为`true`，开启后在正文中插入如下代码即可

    <div class="github-widget" data-repo="<GitHub 用户名>/<仓库名>.js"></div>
    
    <!-- e.g. -->
    <div class="github-widget" data-repo="xiaweizi/QNews"></div>
    
![GitHub 小挂件](http://upload-images.jianshu.io/upload_images/4043475-18522728a2540610.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个小挂件展示了对应仓库的链接地址、`star`和`fork`的数量等等信息。

至于其他的三方服务，只要按照官方的配置都可以完成，我就不赘述了，在配置的过程中遇到什么问题，咱们可以共同交流！

快过年了，今天也是情人节，祝撸友们新年快乐，新年脱单！

![新年快乐](http://upload-images.jianshu.io/upload_images/4043475-9edf8267daf8b3a5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
