---
title: 简单粗暴----RecyclerView
date: '2017.01.06 20:46:25'
categories:
  - 技术分享
tags:
  - Material Design
abbrlink: 7798
---

###RecyclerView初步使用，下拉刷新，点击事件
***
###本篇文章只是给初次使用RecyclerView的兄弟一个简单的入门使用步骤。

<!-- more -->

![RecyclerView预览](http://upload-images.jianshu.io/upload_images/4043475-7ff5227d2f9e52f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###现如今，RecyclerView已经逐渐开始代替 ListView 和 GridView,只需要一步，就可以实现ListView 和 GridView 之间的切换，后面会叙述。当我们还在为ListView优化烦恼的时候，RecyclerView已经帮你封装好了，使用起来也很简单。
>***先看一下运行结果***

![](http://upload-images.jianshu.io/upload_images/4043475-07f75443b572daa6.gif?imageMogr2/auto-orient/strip)
## 1.初步使用  ##
##### 1. 布局添加控件 ##
        <android.support.v7.widget.RecyclerView
            android:id="@+id/rv"
            android:layout_width="match_parent"
            android:layout_height="match_parent">     
        </android.support.v7.widget.RecyclerView>
##### 2. 代码完成设置 ##
        rv = (RecyclerView) view.findViewById(R.id.rv);
        //新建适配器
        adapter = new MyNewsDataAdapter(MainActivity.this);
        //设置布局管理器(下面三种任选其中一个)
        //1. ListView
        rv.setLayoutManager(new LinearLayoutManager(
                MainActivity.this,                //上下文
                LinearLayoutManager.VERTICAL,    //设置布局方向(垂直）
                false));                        //是否翻转
        //2. GridView
        rv.setLayoutManager(new GridLayoutManager(
                MainActivity.this,                //上下文
                3,                                //列或者行的数量
                LinearLayoutManager.VERTICAL,    //布局方向
                false                            //是否翻转
        ));
        //3. 瀑布流
        rv.setLayoutManager(new StaggeredGridLayoutManager(
                3,                                //如果方向垂直则代表列，否则代表行
                StaggeredGridLayoutManager.VERTICAL//布局方向
        ));
        //设置适配器
        rv.setAdapter(adapter);
        //添加数据到适配器中，(这个方法在适配器中)
        adapter.addDataToAdapter(dataBeens, true);
##### 3. 剩下就是适配器的设置##
    1. 新建类继承自RecyclerView.Adapter<MyNewsDataAdapter.MyViewHolder>，
    此时会报红，那是因为要实现必须实现的方法。但是在实现方法之前，要创建一个继承自
    RecyclerView.ViewHolder的类。在里面声明，item的控件，并进行findViewById，
    找到对应xml中id
    class MyViewHolder extends RecyclerView.ViewHolder {
            public TextView  content;
            public TextView  title;
            public TextView  time;
            public ImageView icon;
            public MyViewHolder (View itemView) {
                super(itemView);
                content = (TextView) itemView.findViewById(R.id.tv_content);
                title = (TextView) itemView.findViewById(R.id.tv_title);
                icon = (ImageView) itemView.findViewById(R.id.iv_icon);
                time = (TextView) itemView.findViewById(R.id.tv_time);    
            }
        } 
    2. 创建构造函数，传入上下文，初始化数据
    public MyNewsDataAdapter (Context context) {
            Log.i(TAG, "MyAdapter: 初始化");
            mDatas = new ArrayList<>();
            this.context = context;
        }
      3. 创建方法传入数据到适配器中
        /**
         * 传入数据到适配器中，并判断是否清除老数据
         * @param datas         传入的数据
         * @param isClearOld    是否清除老数据
         */
        public void addDataToAdapter(List<NewsData.DataBean> datas, boolean isClearOld){
            if (datas != null){
                if (isClearOld){
                    mDatas.clear();
                }
                mDatas.addAll(datas);
            }
        }
     4. 接下来，就是完成所需要实现的方法
        a. getItemCount 返回数据的数量
        @Override
            public int getItemCount () {
                return mDatas.size();
            }
        b. onCreateViewHolder 主要是自定义ViewHolder的初始化，并返回出去
         @Override
            public MyViewHolder onCreateViewHolder (ViewGroup parent, int viewType) {
                View         view   = View.inflate(context, R.layout.item_news_data, null);
                MyViewHolder holder = new MyViewHolder(view);        
                return holder;
            }
        c. onBindViewHolder 这里就是给控件设置值
        @Override
            public void onBindViewHolder (MyViewHolder holder, final int position) {        
                  //获取数据
                NewsData.DataBean dataBean = mDatas.get(position);
                String            title    = dataBean.getTitle();
                String            summary  = dataBean.getSummary();
                String            iconUrl  = dataBean.getIcon();
                String            stamp    = dataBean.getStamp();
                //加载数据
                holder.content.setText(summary);
                holder.title.setText(title);
                holder.time.setText(stamp);
                /*使用Glide框架加载图片*/
                Glide.with(context).load(iconUrl).into(holder.icon);
            }
###这个时候就大功告成了看一下运行结构，使用垂直ListView，其他方式就自己尝试把。
![](http://upload-images.jianshu.io/upload_images/4043475-0de9ab408fee4f6a.gif?imageMogr2/auto-orient/strip)
>适配器里面代码虽然看起来有点多，但是只要使用过ListView的肯定能看懂，并且可以根据自己需求进行代码的删减。

## 2.RecyclerView下拉刷新  ##
###使用(SwipeRefreshLayout + RecyclerView)方式实现简单的下拉刷新
##### 1. 在布局添加##
    <android.support.v4.widget.SwipeRefreshLayout
            android:id="@+id/srl"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">
            <android.support.v7.widget.RecyclerView
                android:id="@+id/rv"
                android:layout_width="match_parent"
                android:layout_height="match_parent">                                  
             </android.support.v7.widget.RecyclerView>
    </android.support.v4.widget.SwipeRefreshLayout>
##### 2. 在代码添加##
    srl = (SwipeRefreshLayout) view.findViewById(R.id.srl);
    
    srl.setColorSchemeColors(Color.RED, Color.BLUE, Color.GREEN, Color.YELLOW);//设置进度框颜色的切换
    srl.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
        @Override
        public void onRefresh () {

            srl.setRefreshing(false);//取消进度框
            Toast.makeText(getActivity(), "刷新成功", Toast.LENGTH_SHORT).show();
        }
    });
>就这么简单，来看一下效果

![](http://upload-images.jianshu.io/upload_images/4043475-6e5adf37203df250.gif?imageMogr2/auto-orient/strip)

## 3.RecyclerView点击事件  ##
>RecyclerView是没有像ListView那样的点击事件的。啥？！！这个时候别慌，我们完全可以自己设置，正好顺便复习一下 **java的回调机制**

##### 1. 在适配器中创建一个接口，定义两个方法 ##
     public interface OnRecyclerViewItemClickLisetener {            void onItemClick(View view, int position);
            void onItemLongClick(View view, int position);
     }
##### 2. 在适配器中定义私有监听器##
    private OnRecyclerViewItemClickLisetener mListener = null;
##### 3. 在onBindViewHolder()方法中，给每个itemView设置点击事件，当itemView被点击的时候，回调自己定义接口，并将位置也传过去，类似，listview的onItemClickListener.##
    holder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick (View v) {
                    mListener.onItemClick(v, position);
                }
            });
    holder.itemView.setOnLongClickListener(new View.OnLongClickListener() {
        @Override
        public boolean onLongClick (View v) {
            mListener.onItemLongClick(v, position);
            return true;
        }
    });
##### 4. 暴露一个方法，给使用者设置监听器 ##
    public void setOnClickListener(OnRecyclerViewItemClickLisetener listener){
            mListener = listener;
     }
##### 5. 最后使用就简单了
    adapter.setOnClickListener(new MyNewsDataAdapter.OnRecyclerViewItemClickLisetener() {
        @Override
        public void onItemClick (View view, int position) {
            Toast.makeText(MainActivity.this, position + "位置" + "被点击了", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onItemLongClick (View view, int position) {
            Toast.makeText(MainActivity.this, position + "位置" + "被长时间点击了", Toast.LENGTH_SHORT).show();
        }

     });
###看一下结果
![](http://upload-images.jianshu.io/upload_images/4043475-737148dd65549f7e.gif?imageMogr2/auto-orient/strip)
