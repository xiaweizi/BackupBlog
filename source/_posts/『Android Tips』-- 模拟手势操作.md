---
title: 『Android Tips』—— 模拟手势操作
author: 下位子
tags:
  - 工具
  - 工作积累
categories:
  - Android tips
abbrlink: 496f
date: 2018-05-28 22:11:12
---

> 平时 `Android` 开发中总会遇到奇葩的功能或者需求，这里做个记录和积累，以便后面开发过程中遇到类似的问题，可以快速的解决。[Android tips](http://xiaweizi.cn/categories/Android-tips/)

## 前言

这个版本终于快结束了，历时一个月的时间，这段时间里重复着开发、找 `BUG` 和解 `BUG` 的工作，人已经快麻木了，不过最后看到 自己的开发成果还是蛮欣慰的，这可能就是程序员最简单的乐趣吧。这里看一下整体的效果图，一些细节不方便展示，大概有个预览吧：

<!-- more -->

![整体交互](http://xiaweizi.top/2018-05-29-jiaohu.gif)

里面有很多细小的知识点，比如阴影、状态栏颜色、下拉刷新、tab 悬浮、TCP 更新、数据缓存等等，或者是奇葩的 `BUG`，比如透明主题的 `Activity` 下 `Dialog` 的展示、`Fragment` 的 `setUserVisibleHint`没有按照预期执行等等，总之，在这次版本过后，感觉自己写 `UI`的能力提高了不少。

今天我介绍整个版本中一个非常小的细节处理，并对这个功能进行了稍微的扩展，希望能够帮到你们。

## 简介

进入正题，整个界面我是通过一个通用的下拉刷新控件 + `design` 库 `CoordinatorLayout` 实现的。那这里有个小需求，看一下效果：

![card](http://xiaweizi.top/2018-05-29-TouchPreview.gif)



![head](http://xiaweizi.top/2018-05-29-093648.png)

说明：点击蓝色区域，展开资金页，同时整个界面向下滑动至初始位置。

功能简单，但是实现的过程发现，我外层滑动的 `View` 并不是 `RecyclerViedw` 或者是 `ListView` 更或者是 `ScrollView`，因此并没有相应的接口类似 `smoothScrollToPosition()` ，供我调用。只能自己想办法实现这个效果。

第一：研究 `CoordinatorLayout` 源码，搞懂他的滑动机制，这个对于我来说难度有点大，况且需求的预估时间也不允许我这样做。

第二：另辟蹊径，既然滑动是人为触发的，那就模拟手指滑动事件，让父 `View`下发滑动事件，让子 `View` 接收这个事件并处理。

## 实现

既然找到实现的思路了，那就动手来实现吧。在此之前需要对事件的分发机制有一定的了解，相信大部分小伙伴应该都很熟悉，不过可能也有的人没怎么接触这块，那我做个大致的介绍为后文进行铺垫。

### 事件分发

搞懂事件分发也不难，只要搞懂事件的本质、操作的对象和传递的过程，脑海里就会对此有个基本的概念。

**什么是事件？**

当你接触到屏幕便会产生事件，`Android`系统将其封装成 `MotionEvent`。

|           type            |     desc     |
| :-----------------------: | :----------: |
|  MotionEvent.ACTION_DOWN  |   按下屏幕   |
|   MotionEvent.ACTION_UP   | 从屏幕上移开 |
|  MotionEvent.ACTION_MOVE  | 在屏幕上滑动 |
| MotionEvent.ACTION_CANCEL |  非人为取消  |

**事件在哪传递呢？**

|          type           |              desc              |
| :---------------------: | :----------------------------: |
|        Activity         |         事件传递的起点         |
|        ViewGroup        | 负责接收上层事件、处理或者下发 |
| MotionEvent.ACTION_MOVE |     负责接收上层事件和处理     |

**事件传递的方向是什么？**

从 `Acitivity` -> `ViewGroup` -> `View` 一次传递，最终被处理或者被回传到最上层。

**如何控制事件的传递？**

主要有三个方法决定：

|          type           |      desc      |
| :---------------------: | :------------: |
|  dispatchTouchEvent()   |  负责分发事件  |
| onInterceptTouchEvent() |  负责拦截事件  |
|     onTouchEvent()      | 负责事件的处理 |

**事件传递的流程是什么？**

![事件分发](http://xiaweizi.top/2018-05-29-%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91.png)

上图来源于 [事件分发机制详解](https://www.jianshu.com/p/38015afcdb58)，特此感谢。

整体大概流程就这样，因为篇幅重点不在这，就不做多阐述，我写的可能有点粗糙，如果想看详细的介绍，推荐上方的事件分发机制详解文章。

### 开搞

既然对事件分发有了大致的了解，那接下来就好办了，咱们可以直接构造这个 `MotionEvent` 然后让 `View` 处理，或者让 `Activity` 进行向下分发。

**模拟点击**

首先从简单的开始，模拟手势的点击操作。**点击操作由一个 `Down` 和 `Up` 组合而成**。

```java
MotionEvent downEvent = MotionEvent.obtain(SystemClock.uptimeMillis(), SystemClock.uptimeMillis(), MotionEvent.ACTION_DOWN, x, y, 0);
        MotionEvent upEvent = MotionEvent.obtain(SystemClock.uptimeMillis(), SystemClock.uptimeMillis(), MotionEvent.ACTION_UP, x, y, 0);
// 事件构造出来，接下来就是处理和分发了。
if (object instanceof View) {
    ((View) object).onTouchEvent(downEvent);
    ((View) object).onTouchEvent(upEvent);
    } else if (object instanceof Activity) {
    ((Activity) object).dispatchTouchEvent(downEvent);
    ((Activity) object).dispatchTouchEvent(upEvent);
}
// 记得将事件回收
downEvent.recycle();
upEvent.recycle();
```

我这边封装成了一个静态方法，直接调用即可：

```java
public static void simulateClick(View view, float x, float y) {
        dealSimulateClick(view, x, y);
}
public static void simulateClick(Activity activity, float x, float y) {
    dealSimulateClick(activity, x, y);
}
```

看一下 `Demo` 的运行效果：

![模拟点击](http://xiaweizi.top/2018-05-29-%E6%A8%A1%E6%8B%9F%E7%82%B9%E5%87%BB.gif)

**模拟滑动**

那滑动即一个 `Down` 、一个 `Up` 和 多个 `Move` 事件组成，为了添加一个滑动的延迟效果，使用 `Handler` 来完成。

```java
Handler handler;
if (object instanceof View) {
    View view = (View) object;
    handler = new ViewHandler(view);
    view.onTouchEvent(MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, startX, startY, 0));
    GestureBean bean = new GestureBean(startX, startY, endX, endY, duration, period);
    Message.obtain(handler, 1, bean).sendToTarget();
} else if (object instanceof Activity) {
    Activity activity = (Activity) object;
    handler = new ActivityHandler(activity);
    activity.dispatchTouchEvent(MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_DOWN, startX, startY, 0));
    GestureBean bean = new GestureBean(startX, startY, endX, endY, duration, period);
    Message.obtain(handler, 1, bean).sendToTarget();
}
```

在 `Handler` 里不断的发送 `Move` 事件，完成模拟手指滑动的效果。

```java
static class ActivityHandler extends Handler {
    WeakReference<Activity> mActivity;

    ActivityHandler(Activity activity) {
        super(Looper.getMainLooper());
        mActivity = new WeakReference<>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
        Activity theActivity = mActivity.get();
        if (theActivity == null || theActivity.isFinishing()) {
            return;
        }
        long downTime = SystemClock.uptimeMillis();
        GestureBean bean = (GestureBean) msg.obj;
        long count = bean.count;
        if (count % 10 == 0) {
            Log.i(TAG, "handleMessage: " + count);
        }
        if (count >= bean.totalCount) {
            theActivity.dispatchTouchEvent(MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_UP, bean.endX, bean.endY, 0));
        } else {
            theActivity.dispatchTouchEvent(MotionEvent.obtain(downTime, downTime, MotionEvent.ACTION_MOVE, bean.startX + bean.ratioX * count, bean.startY + bean.ratioY * count, 0));
            bean.count++;
            Message message = new Message();
            message.obj = bean;
            sendMessageDelayed(message, bean.period);
        }
    }
}
```

我也封装了一个静态方法进行使用：

```java
    /**
     * 模拟手势滑动
     *
     * @param activity 当前的 activity
     * @param startX   起始位置 x
     * @param startY   起始位置 y
     * @param endX     终点位置 x
     * @param endY     终点位置 y
     * @param duration 滑动时长 单位 ms
     * @param period   滑动周期
     *                 {@link #LOW} 慢
     *                 {@link #NORMAL} 正常
     *                 {@link #HIGH} 高
     */
    public static void simulateScroll(Activity activity, float startX, float startY, float endX, float endY, long duration, int period) {
        dealSimulateScroll(activity, startX, startY, endX, endY, duration, period);
    }
```

看一下 `Demo`  的运行效果：

![模拟滑动](http://xiaweizi.top/2018-05-29-%E6%A8%A1%E6%8B%9F%E6%BB%91%E5%8A%A8.gif)

模拟手势画了一个⭐️，具体的源码我已经上传到了 `Github` 上 [ScrollDemo](https://github.com/xiaweizi/ScollDemo)。

## 总结

从一个非常小的需求实现细节就可以拓展很多知识点，不得不说这个版本学了很多东西，或许这次分享可以帮到你。