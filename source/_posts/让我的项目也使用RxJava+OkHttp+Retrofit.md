title: jsoup爬虫简书首页数据做个小Demo
date: 2017.02.25 20:46:25
categories:
- 技术分享
tags:
- 第三方库
- 项目
---

# 篇前唠叨 #
*[我的博客地址](http://xiaweizi.cn/)*
之前写了一个小项目 [趣闻](https://github.com/xiaweizi/QNews) ，是我找工作期间自己做的一个小Demo。花了一点时间完成后，便在简书上与大家分享 [快毕业了，撸一个小项目(新闻段子客户端)](http://www.jianshu.com/p/ae4aa11f35a4),竟然在一天之间收到那么多的喜欢：

<!-- more -->

![thank.png](http://upload-images.jianshu.io/upload_images/4043475-dd870b8e27244fb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这让我这个小菜鸟是万万没有想到的，所以在这我要感谢所有给我鼓励，给我建议的撸友们，谢谢你们，我会努力的！

*之前项目用的是 OkHttp 的开源框架完成的网络请求，但是我们程序员要跟得上时代的潮流!*
![rank.png](http://upload-images.jianshu.io/upload_images/4043475-67703efffc224725.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*可以发现 RxJava 和 Retrofit 位居排行榜的第一第三，由此可见学习RxJava+Retrofit的必要性！*

于是，我花了两天的时间学习了一下 Retrofit 和 RxJava，发现RxJava 真TM强呀！相见恨晚哪！

因为花的时间不多，所以只了解到一些皮毛，然后趁热打铁，尝试着在自己的项目中加入这个强大的框架，有些细节可能处理的不好，请多多指教！

>先声明：本篇文章只介绍如何简单的使用 Retrofit+RxJava, 详细部分就请大家自己查找资料，因为本人知道也太少...

>推荐觉得比较好的文章：

>- [扔物线](https://github.com/rengwuxian) 的 [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
>- [RxJava 与 Retrofit 结合的最佳实践](https://gank.io/post/56e80c2c677659311bed9841)
>- [Retrofit使用详解系列](http://www.jianshu.com/p/9a5233bc1da8)
>- [手把手教你使用 RxJava 2.0系列](http://www.jianshu.com/p/d149043d103a)

-----

>首先将需要添加的依赖全部添加，后面就不赘述了。
>    
	//Retrofit
    compile 'com.squareup.retrofit2:retrofit:2.2.0'
	//Retrofit通过GSON将结果转换为Bean对象
    compile 'com.squareup.retrofit2:converter-gson:2.0.2'
	//让Retrofit支持RxJava
    compile 'com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0'
    //RxJava
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
    compile 'io.reactivex.rxjava2:rxjava:2.0.1'
    //日志拦截器
    compile 'com.squareup.okhttp3:logging-interceptor:3.4.1'

还是先来一波结果演示图：

![运行结果.gif](http://upload-images.jianshu.io/upload_images/4043475-72b7868b0f671268.gif?imageMogr2/auto-orient/strip)

我就拿我的项目中[段子](https://www.juhe.cn/docs/api/id/95)模块来进行修改，数据还是来源 [聚合数据](https://www.juhe.cn/docs/index/sortby/4)。
# 1. 创建 javaBean 对象
> 我是从 [聚合数据](https://www.juhe.cn/docs/index/sortby/4) 这个平台获取到数据，当你从这个平台申请到这个服务，他会给你一个接口和Appkey，然后通过查询参数可以获取相应的数据。

>这里我就把我的 `Appkey` 无私的贡献出来

>这是我的数据接口 [http://japi.juhe.cn/joke/content/text.from?key=ae240f7fba620fc370b803566654949e&page=1&pagesize=10](http://japi.juhe.cn/joke/content/text.from?key=ae240f7fba620fc370b803566654949e&page=1&pagesize=10)

>相信大家都能看的懂吧，除了固定 `url`，后面参数分别代表 开发者的 `AppKey`、获取数据的页数和每页显示的条数。

这是我从服务器获取到的 `json` 数据：

	{
	    "error_code": 0,
	    "reason": "Success",
	    "result": {"data": [{
	        "content": "一男子跟朋友走在路上，突然发现五个女人正在围殴其丈母娘！　　朋友见状，忙问男子：\u201c你还不赶紧上去帮忙？！\u201d　　男子镇定地说：\u201c不用，五个人足够了！\u201d",
	        "hashId": "0a8b6fdd70daa311aa82a771132d9895",
	        "unixtime": 1487980430,
	        "updatetime": "2017-02-25 07:53:50"
	    }]}
	}

然后通过 `Android Studio` 插件 `GsonFormat` 自动生成 `javaBean` 对象，这里我就不把代码贴出来了，太多了。 

# 2. 创建服务接口和获取数据的方法 #

	public interface QNewsService {
	
	    public static final String DESC = "desc"; // 指定时间之前发布的
	    public static final String ASC = "asc";   // 指定时间之后发布的
	
	
	    /**
		 * 查询最新的段子数据
		 * 
	     * @param page      查询的页数
	     * @param pagesize  一页数据显示的条数
	     * @return          查询结束返回的被观察者
	     */
		//接口完整地址
	    // http://japi.juhe.cn/joke/content/text.from?key=ae240f7fba620fc370b803566654949e
	    @GET("text.from?key=ae240f7fba620fc370b803566654949e")
	    Observable<JokeBean> getCurrentJokeData(
	            @Query("page") int page,
	            @Query("pagesize") int pagesize
	    );
	
	
	    /**
		 * 根据指定的时间，获取相关的段子数据
		 * 
	     * @param time          要指定查询的时间
	     * @param page          查询的页数
	     * @param pagesize      一页数据显示的条数
	     * @param sort          判断是在指定时间之前还是之后
	     *                          {@value DESC 指定之前},{@value ASC 指定之后}
	     * @return              查询结束返回的被观察者
	     */
		//接口完整地址
	    // http://japi.juhe.cn/joke/content/list.from?key=ae240f7fba620fc370b803566654949e&page=1&pagesize=5&sort=desc
	    @GET("list.from?key=ae240f7fba620fc370b803566654949e")
	    Observable<JokeBean> getAssignJokeData(
	            @Query("time") long time,
	            @Query("page") int page,
	            @Query("pagesize") int pagesize,
	            @Query("sort") String sort
	    );
	
	}

> `@GET`： 请求方法为 `GET`

> `("text.from?key=ae240f7fba620fc370b803566654949e")`：除去 `baseUrl`，因为 `AppKey` 值固定，就可以直接写进去，要查询的单独列出来

> `Observable<JokeBean>`:这个就是 `RxJava+Retrofit` 的简单结合体，当你从服务器获取数据直接返回JavaBean对象，是不是很强大，然后把它有转换成 被观察者，方便后面事件的传递。

> `@Query("page") int page`：通过 `@Query` 来设置查询条件

# 3. 创建客户端 #
	public class QClitent {
	
	    private static QClitent mQClient;
	    private Retrofit mRetrofit;
	
	    private QClitent(){
	        mRetrofit = createRetrofit();
	    }
	
	    public static QClitent getInstance() {
	        if (mQClient == null){
	            synchronized (QClitent.class){
	                if (mQClient == null){
	                    mQClient = new QClitent();
	                }
	            }
	        }
	        return mQClient;
	    }
	    /**
     	* 创建相应的服务接口
     	*/
	    public <T> T create(Class<T> service){
	        checkNotNull(service, "service is null");
	        return mRetrofit.create(service);
	    }
	    /**
	     * 直接创建 QNewsService
	     */	
	    public QNewsService createQNewsService(){
	        return mRetrofit.create(QNewsService.class);
	    }
	
	    private  <T> T checkNotNull(T object, String message) {
	        if (object == null) {
	            throw new NullPointerException(message);
	        }
	        return object;
	    }
	
	    private Retrofit createRetrofit() {
	        //初始化OkHttp
	        OkHttpClient.Builder builder = new OkHttpClient.Builder()
	                .connectTimeout(9, TimeUnit.SECONDS)    //设置连接超时 9s
	                .readTimeout(10, TimeUnit.SECONDS);      //设置读取超时 10s
	
	        if (BuildConfig.DEBUG) { // 判断是否为debug
	            // 如果为 debug 模式，则添加日志拦截器
	            HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
	            interceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
	            builder.addInterceptor(interceptor);
	        }
	
	        // 返回 Retrofit 对象
	        return new Retrofit.Builder()
	                .baseUrl(Constants.BASE_JOKE_URL)
	                .client(builder.build()) // 传入请求客户端
	                .addConverterFactory(GsonConverterFactory.create()) // 添加Gson转换工厂
	                .addCallAdapterFactory(RxJava2CallAdapterFactory.create()) // 添加RxJava2调用适配工厂
	                .build();
	
	    }
	}

> 这里采用 单例模式 获取 `Retrofit` 对象，创建相应的服务接口。

> 注释写的比较详细，我就不多说了。

# 4. 使用 RxJava 完成异步请求#
    private void updateDate() {
        srlJoke.setRefreshing(true);    // 让SwipeRefreshLayout开启刷新
        QClitent.getInstance()
                .create(QNewsService.class) // 创建服务
                .getCurrentJokeData(1, 8)   // 查询
                .subscribeOn(Schedulers.io())   //  指定被观察者的操作在io线程中完成
                .observeOn(AndroidSchedulers.mainThread())  // 指定观察者接收到数据，然后在Main线程中完成
                .subscribe(new Consumer<JokeBean>() {
                    @Override
                    public void accept(JokeBean jokeBean) throws Exception {
                        // 成功获取数据
                        mAdapter.setNewData(jokeBean.getResult().getData());    // 给适配器设置数据
                        srlJoke.setRefreshing(false);       // 让SwipeRefreshLayout关闭刷新
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        // 获取数据失败
                        Toast.makeText(getActivity(), "获取数据失败", Toast.LENGTH_SHORT).show();
                        srlJoke.setRefreshing(false);
                    }
                });
    }

> `RxJava` 在 `GitHub` 主页上的自我介绍是` "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"`
> （一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。这就是 RxJava ，概括得非常精准。

> 通过强大的链式调用，一目了然，简单迅速的完成了网络的异步请求和数据加载。这就是 `RxJava+Retrofit` 的强大之处！其实这只是一小部分，`RxJava` 的功能远不止于此，强大的操作符基本上可以完成项目中所有事件传递的需要，推荐大家，一定好好研究一下这个强大的框架！

最后的结果就是这样，跟之前没什么太大区别：

![运行结果.gif](http://upload-images.jianshu.io/upload_images/4043475-72b7868b0f671268.gif?imageMogr2/auto-orient/strip)

好了，到此，简单的 `RxJava+Retrofit` 小应用就结束了，虽然目前还是没有找到工作，但是丝毫不影响我继续学习下去，我相信，肯定有一家公司愿意收留我的！
