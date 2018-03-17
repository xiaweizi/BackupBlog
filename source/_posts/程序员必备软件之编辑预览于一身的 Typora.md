---
title: 程序员必备软件之编辑预览于一身的 Typora
author: 下位子
tags:
  - 工具软件
  - 提高效率
categories:
  - 程序员必备软件
abbrlink: typora
date: 2018-03-17 16:16:12

---

## 前言

作为程序员，相信对 `MarkDown` 语法并不陌生，平时知识的积累，博客的编写或者是工作的报告都或多或少会用到。

> `Markdown` 是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

那具体的介绍和使用语法就不用过多介绍，较为简单，网上一搜一大把。对应的编辑器也很多，比如之前使用的 `MacDown`，或者是笔记软件，更或者是博文平台都是支持 `MarkDown` 语法的。就我使用的过程来看，大部分的界面都是编辑+预览的，那 `Typora` 不同于其他的编辑工具，当输入相应的标记符号，系统便会自动渲染文本，形成相对应的格式。因此就达到了 **编辑与预览** 同一界面的效果。

<!-- more -->

看一下大概的效果，后面会一一介绍：

![大概效果](/Users/xiaweizi/Desktop/pic/2018-03-17-Typora%E9%A2%84%E8%A7%88%E6%95%88%E6%9E%9C.png)功能还有很多，确实值得拥有。

## 获取

[*Typora* — a markdown editor, markdown reader.](https://www.baidu.com/link?url=KYhuNjN8103k0Uz6EMLJnj07U89iX26TMslP5Yf1CUm&wd=&eqid=8aea3c1900083620000000035aacb943)

进入官网可进行下载，目前 `Mac`、 `Windows` 和 `Linux` 都有对应的版本，当然也有详细的使用介绍。

![mage-20180317145000](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-image-201803171450002.png)

##  基本使用

`Typora` 支持原生的标记语法使用，也支持非常强大的快捷键使用，两者配合着使用，很大程度上可以提高工作效率。

#### 标题

![标题](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E6%A0%87%E9%A2%98.gif)

#### 特殊样式

![特殊样式](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E7%89%B9%E6%AE%8A%E6%A0%B7%E5%BC%8F.gif)

#### 列表

![列表](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E5%88%97%E8%A1%A8.gif)

#### 代码

![代码](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E4%BB%A3%E7%A0%81.gif)

#### 表格

![表格](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E8%A1%A8%E6%A0%BC.gif)

#### 流程图 时序图

![流程图](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E6%B5%81%E7%A8%8B%E5%9B%BE.gif)

相关对应的代码:



```
// 流程图 flow
st=>start: Start
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op

// sequence
Title:连接建立的过程
客户主机->服务器主机: 连接请求（SYN=1,seq=client_isn） 
服务器主机->客户主机: 授予连接（SYN=1,seq=client_isn）\n ack=client_isn+1
客户主机->服务器主机: 确认（SYN=0,seq=client_isn+1）\nack=server_isn+1

// mermaid
graph TD
client1-->|read / write|SVN((SVN server))
client2-->|read only|SVN
client3-->|read / write|SVN
client4-->|read only|SVN
client5(...)-->SVN
SVN---|store the data|sharedrive

```



## 进阶使用

#### 插入图片

有人说插入图片凭什么算作为 **进阶使用**，不就一行代码的事情吗 `![]()`，但是我要说的可不一样。

我们平常需要插入文章应该怎么办？

1. 直接从网络上获取图片的链接地址，作为自己的图片链接(万一链接改变，图片便显示不出)
2. 直接本地文件的相对路径(万一博客需要共享，自然别人看不到图片)
3. 先通过别的平台上传图片(七牛云)，然后拷贝链接地址(操作较为麻烦)
4. 当然可以直接在简书上直接拖拽文件到编辑见面(那又何必用 **Typora**。。。)

有图床神器 **[iPic](https://itunes.apple.com/cn/app/id1101244278?mt=12)**，可以通过拖拽、快捷键等方式上传图片，支持微博、七牛、又拍、阿里云、Imgur、Flickr、Amazon S3 等图床，自动保存 Markdown 格式链接，给你前所未有的插图体验。

看一下效果:

![效果](https://ww4.sinaimg.cn/large/006tKfTcgy1fewqw208xmg30j60aske8.gif)

或者是直接拷贝粘贴图片到编辑器中也可以：

![截图拷贝](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E6%8F%92%E5%85%A5%E5%9B%BE%E7%89%87.gif)



还可以支持多文件上传哦，具体教程请见 [ipic使用教程](https://www.toolinbox.net/iPic/)

#### 版本回溯

我觉得这是它最牛逼的地方了！有点点类似开发过程中的代码回退，比如你想回到某个版本，通过 `git reset [commit]` 即可回到需要的版本，那 **Typora** 的效果如何呢？



![mage-20180317155840](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-image-201803171558408.png)

首先点击 **浏览所有版本**，即可以看到历史的版本：

![mage-20180317160334](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-image-201803171603346.png)

#### 主题更换

**Typora** 支持各种主题的更换

![mage-20180317160756](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-image-201803171607569.png)

![主题更换](http://owj4ejy7m.bkt.clouddn.com/2018-03-17-%E4%B8%BB%E9%A2%98%E6%9B%B4%E6%8D%A2.gif)

## 总结

**Typora** 用习惯了后，真的是离不开他，不仅界面相当友好，而且快捷键功能强大，更提供了方面的插入图片方式，拥有并学会它，一定可以帮助您提高开发效率的。