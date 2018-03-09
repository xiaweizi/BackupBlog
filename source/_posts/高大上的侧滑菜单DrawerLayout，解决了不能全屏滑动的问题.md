---
title: 高大上的侧滑菜单DrawerLayout，解决了不能全屏滑动的问题
date: '2017.01.07 20:46:25'
categories:
  - 技术分享
tags:
  - Material Design
abbrlink: 7455
---

#### 自从2014那年谷歌提出的Material Design后，这种设计语言就广泛被程序猿使用，屡试不爽。在现如今的各个流行APP中，你都能发现它的身影。详细情况，自己百度吧，我只想说很装B。 ##
---

<!-- more -->

####今天我就说一下其中的一个控件 DrawerLayout。在此之前，我一直用的是SlidingMenu，虽然体验也不错，但是也有一些bug...比如不能修改成沉浸式状态栏，后来有位大神跟我说如何修改，但对其印象就不是很好。当我接触到DrawerLayout，发现它用起来简单，方便，毕竟是谷歌亲儿子。

![DrawerLayout预览](http://upload-images.jianshu.io/upload_images/4043475-c6b4df995f7f075c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####DrawerLayout主要功能就是 实现侧滑菜单效果的功能，并且可以通过增加一些设置来实现高大上的效果，那么就请看动态图：

![](http://upload-images.jianshu.io/upload_images/4043475-118894140cac3bca.gif?imageMogr2/auto-orient/strip)

>注意左上角那个图标，有木有很好玩，哈哈...

## 接下来就介绍如何实现这一功能 ##
####1. 在项目对应的build.gradle中添加依赖

        dependencies {
            ...//其他代码
            compile 'com.android.support:appcompat-v7:24.0.0'
            compile 'com.android.support:design:24.0.0'
               ...//其他代码
        }
####2. 添加ToolBar，创建toolbar.xml文件

        <?xml version="1.0" encoding="utf-8"?>
            <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:app="http://schemas.android.com/apk/res-auto"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content">
        
            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:clipToPadding="true"
                android:fitsSystemWindows="true"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:title="资讯"
                app:titleTextColor="#fff">
            </android.support.v7.widget.Toolbar>
        </RelativeLayout>
####3. 在main.xml中添加DrawerLayout    

        <?xml version="1.0" encoding="utf-8"?>
        <LinearLayout
            xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <!-- 添加ToolBar -->
            <include layout="@layout/toolbar"/>

            <!--添加DrawerLayout-->
            <android.support.v4.widget.DrawerLayout
                android:id="@+id/drawerlayout"
                android:layout_width="match_parent"
                android:layout_height="match_parent">

                <!-- 一般第一个位置的代表 主内容 -->
                <FrameLayout
                    android:id="@+id/main"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent">
                </FrameLayout>

                <!-- 左侧菜单(设置layout_gravity 为left) -->
                <RelativeLayout
                    android:id="@+id/left"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:layout_gravity="left">
                </RelativeLayout>

                <!-- 右侧菜单(设置layout_gravity 为right) -->
                <RelativeLayout
                    android:id="@+id/right"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:layout_gravity="right">
                </RelativeLayout>
                
            </android.support.v4.widget.DrawerLayout>
        </LinearLayout>
>DrawerLayout一般分为三个部分 主内容，左侧菜单，右侧菜单
>每个部分的内容自行设置，我是采用Fragment方式设置内容，这里仅供参考

        //新建Fragment，具体内容我就不详细说了
        fragmentMain = new FragmentMain();
        //添加内容，比较简单的
        getSupportFragmentManager().beginTransaction().replace(R.id.main, fragmentMain).commit();

>到此为止已经初步实现了侧滑菜单的功能，来看一下效果

![DrawerLayout初效果.gif](http://upload-images.jianshu.io/upload_images/4043475-ddfb18ab465bd9e8.gif?imageMogr2/auto-orient/strip)

##然后，就是给侧滑按钮添加效果了
####1. 在此之前要进行view的初始化

        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawerlayout);
        toolbar = (Toolbar) this.findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

####2. 通过ActionBarDrawerToggle来完成效果，操作很简单

        mToggle = new ActionBarDrawerToggle(HomeActivity.this, 
                                            mDrawerLayout, 
                                            toolbar, 
                                            R.string.open,
                                              R.string.close);
        mToggle.syncState();
        mDrawerLayout.addDrawerListener(mToggle);

>这样就结束了

##最后就是解决DrawerLayout不能全屏滑动的问题

    private void setDrawerLeftEdgeSize (Activity activity, DrawerLayout drawerLayout, float displayWidthPercentage) {
        if (activity == null || drawerLayout == null) return;
        try {
            // 找到 ViewDragHelper 并设置 Accessible 为true
            Field leftDraggerField =
                    drawerLayout.getClass().getDeclaredField("mLeftDragger");//Right
            leftDraggerField.setAccessible(true);
            ViewDragHelper leftDragger = (ViewDragHelper) leftDraggerField.get(drawerLayout);

            // 找到 edgeSizeField 并设置 Accessible 为true
            Field edgeSizeField = leftDragger.getClass().getDeclaredField("mEdgeSize");
            edgeSizeField.setAccessible(true);
            int edgeSize = edgeSizeField.getInt(leftDragger);

            // 设置新的边缘大小
            Point displaySize = new Point();
            activity.getWindowManager().getDefaultDisplay().getSize(displaySize);
            edgeSizeField.setInt(leftDragger, Math.max(edgeSize, (int) (displaySize.x *
                    displayWidthPercentage)));
        } catch (NoSuchFieldException e) {
        } catch (IllegalArgumentException e) {
        } catch (IllegalAccessException e) {
        }
    }
>直接调用这个方法即可！最后一个参数 传 1，即可实现全屏滑动。如果你想让右侧菜单也是全屏，只要将对应的 "mLeftDragger" 改为 "mRightDragger"。

*[我的博客地址](http://blog.csdn.net/qq_22656383/article/details/54171112)*
再次感谢那些给我提供帮助的文章，博客和人！！我会努力的！！
