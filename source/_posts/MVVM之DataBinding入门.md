---
title: MVVM之DataBinding入门
date: '2017.08.30 20:46:25'
categories:
  - 技术分享
tags:
  - MVVM
  - DataBinding
abbrlink: 41851
---


# MVVM介绍

> 本篇这是基础入门篇，要想看源码剖析请移步到 [DataBinding实现原理探析](http://www.jianshu.com/p/c4f5411cb0ae)

<!-- more -->

`MVVM`框架类似于早期的`MVC`和最热的`MVP`，但是比起这两个更为强势。`MV-VM`相比于`MVP`，其实就是将`Presenter`层替换成了`ViewModel`层，我们都知道，`MVP`的好处就是将逻辑代码从`View`层抽离出来，做到与UI层的低耦合，但是无形中会创造出许多的接口，有些接口很是冗余，不仅如此，当后期修改数据或者添加新的功能还需要修改或是添加接口，很是麻烦。

这个时候`MV-VM`的优势就体现出来了，`ViewModel`层所需要做的完全就是跟逻辑相关的代码，完全不会涉及到UI。当数据变化，直接驱动UI的改变，中间省去了冗余的接口。同时，在`ViewModel`层编写代码中，要求开发者需要将每个方法尽可能的做的功能单一，不与外部有任何的引用或者是联系，无形中提高了代码的健壮性，方便了后期的单元测试。

`DataBinding`其实就是谷歌出台的工具，是实现UI和数据绑定的框架，`View`和`ViewModel`通过`DataBinding`实现单向绑定或双向绑定，做到UI和数据的相互监听，同时开发者的任务分配也就很明确了，负责`ViewModel`的小伙伴完全不用考虑UI如何实现，很大程度上提高了代码的开发效率和后期出问题跟踪的准确性，针对这些好处，采用`MVVM`进行代码开发还是非常有必要的。

# 初步使用
## 1. `module`的`build.gradle`文件加上一行配置代码

    android {
        ...
        dataBinding {
            enabled = true
        }
    }
## 2. 创建布局文件

只需要在之前布局的基础上，外层嵌套 `<layout></layout>`即可。

    <layout
        xmlns:android="http://schemas.android.com/apk/res/android">
    
        <data>
            <variable
                name="student"
                type="com.xiaweizi.bean.Student"/>
            <!-- 这里 type 必须传完整路径，或者用 import 方式也是可以的 -->
            <!--
                <import type="com.xiaweizi.bean.Student"/>
                <variable
                    name="student"
                    type="Student"/>
            -->
        </data>
    
        <!-- 对应之前的XML文件 -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="center_horizontal"
            android:orientation="vertical">
    
        </LinearLayout>
    </layout>

因为`XML`是不支持自定义导包的，所以通过`import`先导包，如果类名相同的话可以通过`alias`进行区分：

    <import type="android.view.View"/>
    <import type="com.xiaweizi.View"
            alias="MyView"/>

    <variable
        name="view1"
        type="View"/>

    <variable
        name="view2"
        type="MyView"/>

这个时候会在`app\build\generated\source\debug\包名`路径下生成对应的`binding`类，命名方式，举个例子最为直接：

    原XML名:activity_main  ----> 生成对应的binding名: ActivityMainBinding
## 3. `Activity`中替换原来的`setContentView()`代码

    ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
    
## 4. 接下来就是关键的`ViewModel`层
#### a. 单向绑定
咱们先从简单的开始，`DataBinding`有个很大的好处就是摒弃原生`findViewById`频繁的遍历视图层和`ButterKnife`的反射，采用的是数组记录每个`view`

    final Object[] bindings = mapBindings(bindingComponent, root, 8, sIncludes, sViewsWithIds);

在`XML`创建一个`TextView`

    <TextView
        android:id="@+id/tv_content"
        android:text="@{student.name}"
        android:layout_width="match_parent"
        android:layout_height="50dp"/>
在代码中通过`binding`直接可以获取到这个`TextView`

    mBinding.tvContent
那么如何实现单向绑定呢？

    Student student = new Student("xiaweizi", 12);
    mBinding.setStudent(student);
这样就可以直接改变`TextView`的值。

`ViewModel`就是简单的数据

    public class Student {
        public String name;
        public int age;
        public Student(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }

#### b. 双向绑定
之前说的单向绑定，即当数据变化，通过`mBinding.setStudent(student)`方式驱动UI的改变
而双向绑定，无论`View`还是`ViewModel`谁改变，都会驱动另一方的改变，实现双向绑定有两种方式：继承`BaseObservable`和使用`ObservableField`创建成员变量。

代码实现：
第一种继承`BaseObservable`：

    public class Student extends BaseObservable{

        // 如果是 public 则在成员变量上方加上 @Bindable 注解
        @Bindable
        public String sex;
    
        public void setSex(String sex) {
            this.sex = sex;
            notifyPropertyChanged(BR.sex);
        }
        
        /*************************** 我是分割线 ***************************/
        // 如果是 private 则在成员变量的 get 方法中添加 @Bindable 注解
        private String name;
        @Bindable
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
            notifyPropertyChanged(BR.name);
        }
        
        public void setSexName(String name, String sex){
            this.name = name;
            this.sex = sex;
            notifyChange();
        }
    }
    
这个时候当调用`setName()`方法,不仅数据改变，UI中的`TextView`内容也会随之改变。

我们可以发现有两个方法：`notifyPropertyChanged()`和`notifyChange`,一个是更新指定的变量，第二个是更新所有该`ViewModel`中的对象。

而`notifyPropertyChanged(int fieldId)`里面传的参数，即上面通过`@Bindable`注解创建对应的变量`id`。

第二种：使用`ObservableField`

    public class Student extends BaseObservable{
    
        public ObservableField<String> name = new ObservableField<>();
        
        private ObservableInt age = new ObservableInt();
        public void setAge(int age) {
            this.age.set(age);
        }
        public int getAge() {
            return age.get();
        }
    }

通过使用`ObservableField`创建的对象作用相当于第一种的方案，支持`ObservableInt`、`ObservableBoolean`或者是`ObservableField<T>`指定的类型、`ObservableArrayMap<String, Object>`、`ObservableArrayList<Object>`等。

`ObservableField`内部已经封装了`get`和`set`方法，如果成员变量是`public`属性，直接通过

    mStudent.name.set("shabi");
    String name = mStudent.name.get();
    
设置和获取对应的成员变量的值。

# 其他使用
学会了上面基本的用户还是远远不够的，像按钮的点击事件或是`EditText`内容的监听，这些也是非常重要的，不过学会了一种，其他的举一反三就会容易的多了。

### 1. 事件处理

`dataBinding`需要你通过一些表达式来处理`view`的分发事件，除了少数例子外，事件元素的名称是由监听器中的方法所控制。比如`View.OnLongClickListener`内部有`onLongClick()`方法，所以`XML`定义的事件就为`android:onLongClick`.

可以直接在`Activity`内部定义一个类，用于处理事件的监听

    public class Presenter {
        public void onClickExample(View view) {
            Toast.makeText(SimpleActivity.this, "点到了", Toast.LENGTH_SHORT).show();
        }
        
        public void onTextChanged(CharSequence s, int start, int before, int count) {
            mStudent.name.set(s.toString());
        }

        public void onClickListenerBinding(Student student) {
            Toast.makeText(SimpleActivity.this, student.name.get(),Toast.LENGTH_SHORT).show();
        }
    }
    
`XML`中:
    
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="输入name"
        android:onTextChanged="@{presenter::onTextChanged}"/>


    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="@{presenter.onClickExample}"
        android:text='@{"年龄：" + student.age}'/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="5dp"
        android:onClick="@{() -> presenter.onClickListenerBinding(student)}"
        android:text='@{"姓名：" + student.name}'/>
        
首先从点击事件开始分析，`android:onClick="@{presenter.onClickExample}"` 里面对应的方法自然是要与`Presenter`定义的方法名一致，名字可以不为`onClickExample`，但是参数必须是`View`，参数要对应于`setOnClickListener(onClickListener listener)`对应的`onClickListener`要实现的接口，即`public void onClick(View)`。

同理，监听`EditText`文本的变化，一般只要注意`onTextChanged(CharSequence s, int start, int before, int count)`方法即可，那么我们可以创建与之对应的方法，在`XML`文件中引用：`android:onTextChanged="@{presenter::onTextChanged}"`。

最后再来看从UI中获取数据，也就是数据的回调，即`DataBinding`的精髓支出，`View`和`ViewModel`双向绑定。`android:onClick="@{() -> presenter.onClickListenerBinding(student)}`这里用到了`lamda`表达式，这样就可以不遵循默认的方法签名，将`student`对象直接传回点击方法中。来看一下实现效果：

![简单测试.gif](http://upload-images.jianshu.io/upload_images/4043475-6355339021fd68cb.gif?imageMogr2/auto-orient/strip)

一目了然，我就不赘述了，我们可以发现一点，一开始我们并没有给`Student`对象设置值，所以显示的是`null`，并没有报空指针异常，这也是`DataBinding`的有点之一。

其实`dataBinding`自带对数据监听的方法：

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@={student.name}"/>
代码中：

    student.addOnPropertyChangedCallback(new Observable.OnPropertyChangedCallback() {
            @Override
            public void onPropertyChanged(Observable observable, int i) {
                // i 为 BR 文件中对应的 int 值
                Log.i("xwz--->", student.getName());
                Log.i("xwz--->", student.getAge());
            }
    });

这个对数据的监听建立在，使用`@Bindable`作为双向绑定为条件，当数据变化，便会出发`onPropertyChanged`方法。需要注意的是`android:text="@={student.name}"`，@后面多了一个`=`。

### 2. `ViewStub`和`include`
`dataBinding`同样是支持`ViewStub`的，使用起来也很简单，直接贴代码了。

    <ViewStub
        android:id="@+id/view_stub"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout="@layout/viewstub"/>
代码中：

    View inflate = binding.viewStub.getViewStub().inflate();

`inflate`即为替代`ViewStub`的`View`.

至于`include`更简单，用法跟以前是差不多，唯一不同的是可以将`ViewModel`传到下一个`XML`中：

    <include layout="@layout/layout_include" bind:student="@{student}"/>

`layout_include`中同样可以共享`student`这个对象。

### 3. `BindingAdapter`的使用
我们之前用的都是`Android`自带的监听或是属性，比如`text`、`onClick`，但是如果项目中需要动态改变`ImageView`的内容，那我们应该怎么办呢？`dataBinding`给我们提供了`BindingAdapter`这个注解，方便我们定义自定义的属性。
假如我们有个需求，点击按钮更换图片，这个时候我们需要定义静态的方法：

    @BindingAdapter({"url", "name"})
    public static void loadImageView(ImageView view, String url, String name) {
        Log.i("xwz--->", url + "\t" + name);
        Glide.with(view.getContext())
             .load(url)
             .into(view);
    }
在`XML`中使用

    <ImageView
        android:layout_width="160dp"
        android:layout_height="160dp"
        bind:name="@{student.name}"
        bind:url="@{student.imgUrl}"/>
这里有必要解释一下，静态方法`loadImageView`里第一个参数为作用的`View`，这里是`ImageView`;后面的参数即分别对应于`@BindingAdapter`里面的参数。那这里是怎么跟`View`联系在一块呢？我们发现`XML`中有这样一行代码`bind:name="@{student.name}`这里的`name`对应的的`@BindingAdapter`注解里的参数`name`，并映射于`ViewModel`中的`student.name`。当`student.name`值改变，就会触发`loadImageView`方法，从而执行里面的方法。

`bind`名称是任意的定义的，不过要定义对应的命名空间`xmlns:bind="http://schemas.android.com/apk/res-auto"`。


实现的效果就很简单了：

![bindAdapter.gif](http://upload-images.jianshu.io/upload_images/4043475-4f03698849422487.gif?imageMogr2/auto-orient/strip)


更强大的在于可以覆盖`Android`原生的元素设置属性，比如`android:text`最常见不过了

    @BindingAdapter ("android:text")
    public static void setText(TextView view, String text) {
        view.setText(text + "xiaweizi");
        Log.i("xwz--->", text);
    }
XML：

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text='@{"测试"}'/>

这个时候所有设置`text`的地方后缀全部加上了`xiaweizi`.

### 4. `@BindingConversion`

`dataBinding`还支持对数据的转换，或者是类型的转换

    @BindingConversion
    public static String addString(String text){
        Log.i("xwz--->", "DemoBindingAdapter:  " + "addString: " + text);
        return text + "xiaweizi";
    }

这个时候会将项目中所有以`@{String}`方式用到的`String`后缀全部加上`xiaweizi`.

    @BindingConversion
    public static ColorDrawable convertColorToDrawable(int color){
       return new ColorDrawable(color);
    }
    
`XML`:

    <View
       android:background="@{isError ? @color/red : @color/white}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>

这段代码的作用在于将`int`类型的`color`值，转换成了`ColorDrawable`类型.

### 5. DataBindingComponent

通过`BindingAdapter`是可以增加一些自定义的属性或者是修改`Android`原生的属性，但是它有一个弊端，就是全局修改所有的相关属性，不过配合上`DataBindingComponent`就可以解决这个问题。

	DataBindingUtil.setContentView(this, R.layout.activity_component, new FirstComponent());

开始的是以一个参数的形式传入这个`View`中，那么只作用于当前的view。

	DataBindingUtil.setDefaultComponent(new FirstComponent());

或者是这种设置全局的方式，也可以改变。

`DataBindingComponent`其实是一个空方法的接口，你需要先创建一个拥有`@BindingAdapter`的类，这里就不能定义为`public`，因为这样`DataBindingComponent`就找不到对应的类，我们为了方便后期的开发，可以定义一个抽象类：

	public abstract class AbstractAdapter {
	    @BindingAdapter ("text")
	    public abstract void setText(TextView textView, String text);
	}

然后定义一个实现类：

	public class FirstAdapter extends AbstractAdapter{
	    @Override
	    public void setText(TextView textView, String text) {
	        Log.i("xwz--->", "FirstAdapter:  " + "setText: ");
	        textView.setText(text+"first");
	    }
	}

这个时候当你创建一个实现`DataBindingComponent`接口的类时，会发现让你实现一个方法：

	public class FirstComponent implements android.databinding.DataBindingComponent {
	    @Override
	    public AbstractAdapter getAbstractAdapter() {
	        return new FirstAdapter();
	    }
	}

这里返回的就是创建的`adapter`，可以根据需求创建对应的`component`.

# RecyclerView中的应用

除了最基本的使用，还有一个频繁出现的就是列表了，那么我们这里就拿`RecyclerView`作为代表进行演示。

`RecyclerView`的好处就不多说了，已经完全代替了之前的`ListView`和`GridView`，用法也就不赘述了，这里主要介绍一下适配器的编写。虽然网上有很多大神已经帮我们创建了各种通用的`adapter`，不过作为入门，我们还是要学习一下使用`dataBinding`创建`adapter`.

先来个简单的`XML`文件：

	<layout xmlns:android="http://schemas.android.com/apk/res/android">
	    <data>
	        <import type="com.github.markzhai.sample.Person"/>
	        <variable
	            name="person"
	            type="Person"/>
	    </data>
	
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="vertical">
	
	        <TextView
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:textSize="24sp"
	            android:padding="5dp"
	            android:textColor="#f0f"
	            android:text='@{"姓名：" + person.name, default="aaa"}'/>
	        <TextView
	            android:layout_width="match_parent"
	            android:layout_height="wrap_content"
	            android:textSize="15sp"
	            android:padding="2dp"
	            android:textColor="#090"
	            android:gravity="right"
	            android:layout_marginRight="80dp"
	            android:text='@{"年龄：" + person.age, default=12}'/>
	    </LinearLayout>
	</layout>

至于`ViewModel`就是一个简单的`Person`类，拥有`name`和`age`两个属性，接下来就是`adapte`的编写。

	public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
	
	    private List<Person> mList;	
	    public MyAdapter() {
	        mList = new ArrayList<>();
	    }
	    public void setData(List<Person> persons) {
	        this.mList = persons;
	    }

	    @Override
	    public MyAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	        ItemRecyclerBinding itemBinding = 
						DataBindingUtil.inflate(LayoutInflater.from(parent.getContext()),
	                                                                  R.layout.item_recycler,
	                                                                  parent,
	                                                                  false);
	
	        return new ViewHolder(itemBinding);
	    }
	
	    @Override
	    public void onBindViewHolder(MyAdapter.ViewHolder holder, int position) {
	        holder.bind(mList.get(position));
	    }
	
	    @Override
	    public int getItemCount() {
	        return mList.size();
	    }
	
	    public static class ViewHolder extends RecyclerView.ViewHolder {
	
	        final ItemRecyclerBinding itemBinding;
	
	        public ViewHolder(ItemRecyclerBinding binding) {
	            super(binding.getRoot());
	            this.itemBinding = binding;
	        }
	
	        void bind(Person person) {
	            itemBinding.setPerson(person);
	        }
	    }
	}

通过`DataBindingUtil.inflate()`创建`item`布局，在`ViewHolder`中进行数据的绑定，这个时候，当数据源变化的时候，`RecyclerView`中的数据也跟着变化。

至于`item`的点击事件可以上面的`onClick`写法：

创建`Presenter`处理点击事件：

    public static class Presenter{
        ItemRecyclerBinding mBinding;
        public Presenter(ItemRecyclerBinding binding){
            this.mBinding = binding;
        }
        public void onItemClick(Person person){
            Log.i("xwz--->", "name: " + person.getName() + "\tage: " + person.getAge());
            Toast.makeText(mBinding.getRoot().getContext(), "name: " + person.getName() + "\tage: " + person.getAge(), Toast.LENGTH_SHORT).show();
        }
    }

在之前的`bind`方法中进行绑定：

    void bind(Person person) {
        itemBinding.setPerson(person);
        itemBinding.setPresenter(new Presenter(itemBinding));
    }

在`XML`中设置点击事件即可：

	<LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onItemClick(person)}"
        android:orientation="vertical">
	...
	</LinearLayout>

这里就可以直接将数据直接回传。

看一下运行效果：

![RecyclerView的使用.gif](http://upload-images.jianshu.io/upload_images/4043475-50a7029e02b7693b.gif?imageMogr2/auto-orient/strip)


# 一些细节

### `databinding`支持一些`java`的表达式

- `+ - * / %`
- 字符串的连接`"a"+"b"`
- 逻辑和位运算`&& || & |`
- 一元运算`+ - ! ~`
- 移位 `>> >>> <<`
- 比较 `== > < >= <=`
- `instance of`
- 支持数据类型:`character,String,numeric,null`
- 强转`cast`
- 方法的调用
- 成员变量的访问
- 数组访问
- 三元表达式`? :`

简单例子：

    android:text="@{String.valueOf(index + 1)}"
    android:visibility="@{age < 13 ? View.GONE : View.VISIBLE}"
    android:transitionName='@{"image_" + id}'
    
### `dataBinding`不支持的`Java`特性

- `this`
- super
- new
- 泛型

### `dataBinding`判空处理

使用`??`来进行判空操作

    android:text="@{user.displayName ?? user.lastName}"

如果不为空则选择左侧值，否则选择右侧值，类似于：

    android:text="@{user.displayName != null ? user.displayName : user.lastName}"
    
### 支持数组，集合，`map`

    <data>
        <import type="android.util.SparseArray"/>
        <import type="java.util.Map"/>
        <import type="java.util.List"/>
        <variable name="list" type="List<String>"/>
        <variable name="sparse" type="SparseArray<String>"/>
        <variable name="map" type="Map<String, String>"/>
        <variable name="index" type="int"/>
        <variable name="key" type="String"/>
    </data>
    …
    android:text="@{list[index]}"
    …
    android:text="@{sparse[index]}"
    …
    android:text="@{map[key]}"
    
### 资源的访问

`dataBinding`支持一般语法对资源的访问：

    android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"



# 小结

`dataBinding`主要的作用就在于减少`Activity`和`Fragment`层的代码,不再使用`findViewById`，让`XML`从之前只用于显示视图，到现在可以做一些操作。在性能上更是有很大的提高，内部采用0反射，使用位标记检测需要更新的`view`，每次数据的改变是在下一帧开始改变等等。

当然也有一些不足之处，`Android Studio`的`IDE`支持还不是那么完善，在`XML`中一些方法不能智能生成和跳转，还有就是报错的错误信息，有的时候并不能定位到准确的位置。不过总体上来说`dataBinding`带来的好处远远的超过这些不足，所以还没有尝试的小伙伴，不妨试一试，相信你会爱上他的。

> 感谢[dataBinding视频](http://www.imooc.com/search/?words=databinding)
> [markzhai](http://blog.zhaiyifan.cn/)
> [官方地址](https://developer.android.google.cn/topic/libraries/data-binding/index.html)

[我的博客](http://xiaweizi.cn)
