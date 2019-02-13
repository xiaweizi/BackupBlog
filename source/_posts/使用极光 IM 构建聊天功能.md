---
title: 『技术分享』-- 使用极光 IM 构建聊天功能
date: '2019.01.19 20:46:25'
categories:
  - 技术分享
tags:
  - 第三方库
abbrlink: '9994'
---


## 前言

距离上次[极光](https://www.jiguang.cn/)征文不知不觉已经过了将近一年的时间，先感谢上次的征文比赛，通过 [《我和 Android 推送的时间简史》](https://www.jianshu.com/p/083d992c3cd1) 这篇文章获奖，这次又厚着脸皮再次参与，因为项目一直很忙，只得利用周末时间准备 demo 素材和写文章，不好之处，多多见谅。

<!-- more -->

上一篇文章主要讲述了 我跟 **极光推送** 的关系，以及简单的描述了其集成和使用，作为三个项目都在使用极光推送的我，对其了解也是相当多的，当然坑也踩了不少，不得不再次感谢大侠(极光技术人员)对我的帮助，虽然这一年没有继续接触推送的业务，但是当我遇到困惑依然有问必答，服务态度不容置疑！

在准备参加征文时就在想应该从哪个角度来写呢，正好之前跟前同事一起写了一个开源项目 [WeaponApp](https://github.com/G-Joker/WeaponApp), 现在已经有 800+ 的 star 了。

![image.png](https://upload-images.jianshu.io/upload_images/4043475-3dd0dcf335cdbc88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

里面涉及的技术我就不一一阐述了，感兴趣的话可以自行看一下，里面有一个模块由我单独负责- **IM模块**，因为已经集成了极光推送，考虑到成本和使用，最终选择了极光IM，毕竟是以极光推送的大规模、高并发、稳定的推送为技术基础，并继承这些特性。那这篇文章就以我集成的经历和使用做个介绍，快速的实现具有 IM 功能的 APP。

## 展示

这里只对 IM 模块功能做简单的演示，感兴趣可以点击 [链接](http://xiaweizi.top/joker-1-1.0.apk) 进行下载，如下 gif 图：

![register.gif](https://upload-images.jianshu.io/upload_images/4043475-a53af219038797aa.gif?imageMogr2/auto-orient/strip)


![ceshi.gif](https://upload-images.jianshu.io/upload_images/4043475-7e5df3865ab51e6b.gif?imageMogr2/auto-orient/strip)

基本的聊天功能已经实现，其中包括：

1. 登录、注册、强踢和退登
2. 个人信息查看和修改
3. 查找好友并进行聊天
4. 群聊
5. 个人中心展示
6. 删除会话和清空聊天记录

后续会根据需要添加新的功能。

## 集成

因为前项已经集成了极光推送服务，很多东西已经不需要重复操作，只需要将 JMessage 相关的组件集成到项目中即可，详情的步骤可直接参考[官网](https://docs.jiguang.cn/jmessage/client/jmessage_android_guide/)。

**1. 导入jmessage jar 包**
**2. 在 AndroidManifest 中添加相应的事件**

没了。。由此可以看出，由推送到 IM 过渡是多么流畅！

## 使用

其实在使用的过程，无非是根据自己的业务需求，到 [官网](https://docs.jiguang.cn/jmessage/client/android_sdk/basic/) 查找 API 来实现自己想要的功能，那我就根据目前项目中有的功能进行介绍。

#### 注册、登录和退登

这其实是用户的信息管理，极光 IM 统一用 `UserInfo` 进行管理，内部包含了用户的大部分信息：

```java
    protected long userID;
    protected String userName;
    protected String nickname = "";
    protected String avatarMediaID;
    protected String birthday = "";
    protected String signature = "";
    protected String gender = "";
    protected String region = "";
    protected String address = "";
    protected String appkey = null;
    protected String notename = null;
    protected String noteText = null;
    protected int mTime;
    protected Map<String, String> extras = new ConcurrentHashMap();
```
**1. 注册**

> 一开始打算自己写用户服务器，事实上却是由另一位开发者完成了，但是考虑到 IM 的集成，用户数据的迁移和转存过程繁琐，就干脆直接用极光的用户接口，其实内部的数据也确实很详细，还支持自定义字段，完全满足日常需求。

```java
JMessageClient.register(userName, password, new JMessageCallBack() {
    @Override
    public void onSuccess() {
        registerSuccess(userName);
    }

    @Override
    public void onFailed(int status, String desc) {
        registerFailed(desc);
    }
});
```
注册需要用户名和密码，注册成功后通过 `setResult` 的方法，将账户和密码传回登录页面。

**2. 登录**

```java
JMessageClient.login(userName, password, new JMessageCallBack() {
    @Override
    public void onSuccess() {
        loginSuccess(userName);
    }

    @Override
    public void onFailed(int status, String desc) {
        loginFailed(desc);
    }
});
```
同注册一样，登录也需要用户名和密码进行登录，如果格式有误会直接触发 `onFailed` 回调，弹出相应的提示。成功后本地便会保存一个 `UserInfo` 对象储存用户的信息。

**3. 退登**

极光支持主动退出账号的功能，即：

```java
JMessageClient.logout();
```

直接清除本地保存的用户信息，同样他支持多端同时在线:

![image.png](https://upload-images.jianshu.io/upload_images/4043475-77639a4f09ce7db6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果不打开开关，另一台设备登录，会用 `EventBus` 发送 `LoginStateChangeEvent`，告知开发者改账号已在另一台设备登录，并且会携带三种状态：

```java
case user_password_change:
	forcedExit("账号密码被修改");
	break;
case user_logout:
	forcedExit("账号在另一台设备登录");
	break;
case user_deleted:
	forcedExit("账号被删除");
	break;
```
根据自己的需要进行处理。

#### 信息查看和修改

**1. 个人**

自己的用户信息其实是保存在本地的数据库中，通过调用:

```java
mUserInfo = JMessageClient.getMyInfo();
```

获取用户信息，之前提过 `UserInfo` 里面包含了用户的所有数据。与之对应的：

```java
JMessageClient.updateMyInfo(UserInfo.Field.gender, mUserInfo, new JMessageCallBack() {
    @Override
    public void onSuccess() {
       
    }

    @Override
    public void onFailed(int status, String desc) {
        
    }
});
```

这个就是修改自己信息的方法，通过传入 `UserInfo.Field` 来区分修改属性值。

**2. 他人**

如果需要查看好友的信息，可通过 `userName` 进行请求查询：

```java
JMessageClient.getUserInfo(userName, new GetUserInfoCallback() {
    @Override
    public void gotResult(int status, String desc, UserInfo userInfo) {
        if (status == 0) {
            getViewModel().setUserInfo(userInfo);
        } else {
            getViewModel().setError(desc);
        }
    }
});
```

具体的结果如下：

![ceshi.gif](https://upload-images.jianshu.io/upload_images/4043475-036a2c3f7678d783.gif?imageMogr2/auto-orient/strip)

如果是个人信息，直接可以修改和退登，如果是他人只能查看并与其进行聊天。

#### 聊天

终于到了核心的聊天功能，其实实现起来并不复杂，极光 IM 已经给了丰富的 API 和使用说明，足够完成基本的需求。

**1. 发消息**

发消息，前提是需要构建 `Message` 对象，以基础文本为例:

```java
final Message message = mConversation.createSendTextMessage(sendContent);
message.setOnSendCompleteCallback(new BasicCallback() {
    @Override
    public void gotResult(int status, String desc) {
        if (status == 0) {
            // 消息发送成功
            MobclickAgent.onEvent(getContext().getApplicationContext(), "send_message", sendContent);
            addSendMessage(message);
            ++curCount;
            setSendContent("");
            getView().scrollToPosition(items.size() - 1);
        } else {
            // 消息发送失败
            Toast.makeText(getContext(), desc, Toast.LENGTH_SHORT).show();
        }
    }
});
JMessageClient.sendMessage(message);
```
最终通过 `JmessageClient.sendMessage(message)` 将消息发送出去。

**2. 接收消息**

他这里比较简单粗暴，直接使用 `EventBus` 进行消息接收的回调。

> 他 jar 里集成了 EventBus，项目中也有了 EventBus，这一点还是蛮坑的。换想一下，这里也是为了方便接收，不然会有很多相互应用，各种耦合，不过使用过 EventBus 的小伙伴，应该知道，如果维护不好 EventBus 会导致逻辑非常的混乱，维护和拓展异常困难。

```java
 * 接收消息事件
 * 目前只支持文字消息，后面再进行优化
 *
 * @param event 消息事件
 */
public void onEventMainThread(MessageEvent event) {
    Message message = event.getMessage();
    switch (message.getContentType()) {
        case text:
            // 处理文字消息
            String userName = ((UserInfo) message.getTargetInfo()).getUserName();
            if (TextUtils.equals(userName, mUserName)) {
                // 当收到的消息是官方消息才进行更新UI
                getViewModel().receiveMessage(message);
            }
        default:
            LogUtils.i("office", message.getFromType());
            break;
    }
}
```

根据 `contentType` 区分消息实体的类型，并做相应的处理。 在需要接收消息的地方进行事件的注册和取消注册。

```java
JMessageClient.registerEventReceiver(this, 200);
JMessageClient.unRegisterEventReceiver(this);
```

**3. 单聊**

这里引入一个 `Conversation` 概念，当你与他人聊天必然会建立会话，那会话的消息和聊天的对象都会存在这个会话中，那单聊则通过传入 `userName` 进行联系，由此可见 `userName` 的唯一性和重要性。

因为刚进去需要获取历史信息，所以通过 `conversion` 获取所有的消息，并展示在界面上。

```java
mConversation = Conversation.createSingleConversation(userName);
JMessageClient.getUserInfo(userName, this);
if (mConversation == null) {
    getView().finish();
}
// 获取本地所有的消息
msgCount = mConversation.getAllMessage().size();
List<Message> messagesFromNewest = mConversation.getMessagesFromNewest(curCount, LIM_COUNT);
curCount = messagesFromNewest.size();
// 第一条消息是正序的，需要反转一下
Collections.reverse(messagesFromNewest);
for (Message message : messagesFromNewest) {
    MessageDirect direct = message.getDirect();
    if (direct == MessageDirect.send) {
        addSendMessage(message);
    } else {
        addReceiverMessage(message);
    }
}
```
**4. 群聊**

群聊与单聊有点类似，不过建立会话的前提参数不是 `userName`， 而是 `groupId` 群的唯一标识 ID。

```java
mConversation = Conversation.createGroupConversation(groupId);
if (mConversation == null) {
    getView().finish();
    return;
}
List<Message> messagesFromNewest = mConversation.getMessagesFromNewest(curCount, LIM_COUNT);
curCount = messagesFromNewest.size();
Collections.reverse(messagesFromNewest);
for (Message message : messagesFromNewest) {
    MessageDirect direct = message.getDirect();
    if (direct == MessageDirect.send) {
        addSendMessage(message);
    } else {
        addReceiverMessage(message);
    }
}
```
具体的代码很相似，只是创建的过程不一样，不再赘述。

## 总结

前段时间王欣、字节跳动等都推出社交软件，不过在微信平台被封杀，这点还是蛮狠的，不过另一方面反映出社交 聊天在各个行业应用的广泛，无论是金融、教育、销售等软件都需要一个 IM 作为用户与用户、用户和平台之间的沟通桥梁，因此作为开发者，还是要多多学习一下 IM 相关的知识。当然自己能独立完成最好，如果没有经历或者暂时能力不够，又或者公司急切需要集成 IM 功能，建议你可以考虑极光 IM 服务，其推送服务做得还是蛮不错的，而且还在不断的维护迭代中，有时间不妨尝试一波吧！

[WeaponApp](https://github.com/G-Joker/WeaponApp)
[APK下载](http://xiaweizi.top/joker-1-1.0.apk)

本文为[极光征文](https://www.jianshu.com/c/41f9ea6ca9b2)参赛文章
