title: 基于LitePal操作数据库的学生管理系统的简单实现
date: 2017.01.22 20:46:25
categories:
- 技术分享
tags:
- 项目
- 第三方库
---

#### 今天看了郭神的《第二行代码》的第六章，才发现LitePal用起来是多么方便简介，就花了下午的时间做了一个小Demo，界面功能都简单，请见谅
---
*本文只是LitePal的简单应用，目的是快速入门LitePal。界面和功能都简单，若不满，请自行添加。*

<!-- more -->

照例来波动态图
![学生管理应用.gif](http://upload-images.jianshu.io/upload_images/4043475-0fff41c510bbc5c6.gif?imageMogr2/auto-orient/strip)
>主要功能就是: 数据的 **增、删、改、查**。
>
>主要知识点: **LitePal的配置和使用**
>
>界面我就不介绍了，用的是RecyclerView之前也讲过 [简单粗暴----RecyclerView](http://www.jianshu.com/p/60819de9eb42)
>
>首先添加依赖 
>
    compile'com.android.support:recyclerview-v7:24.0.0'
    compile 'org.litepal.android:core:1.4.1'

# 1. LitePal简介 #
LitePal是一款开源的Android数据库框架，它采用了对象关系映射（ORM）的模式，并将我们平时开发最常用的一些数据库功能进行了封装，是的不用编写一行SQL语句就可以完成各种建表和增删改查的操作。
# 2. LitePal的配置 #
创建一个 `assets` 目录，在 `assets` 目录下新建一个 `litepal.xml` 文件，接着编写文件内容，如下：

	<?xml version="1.0" encoding="utf-8" ?>
	<litepal>
		<!-- 数据库名 -->
	    <dbname value="Student"></dbname>
		<!-- 版本号 -->
	    <version value="1"></version>
		<!-- 创建表 -->
	    <list>
	        <mapping class="映射的javaBean的完整类名"></mapping>
	    </list>
	</litepal>
接下来修改清单文件代码，配置Application

	<application
        android:name="org.litepal.LitePalApplication"
        ...
    </application>

最后代码中创建数据库

    LitePal.getDatabase();
# 3. 创建表 #
#### a. 需要一个JavaBean对象，也就是数据库的表 ####
---
	public class Student extends DataSupport{

	    private int id;
	    private String name;//姓名
	    private int studentId;//学号
	    private String sex;//性别
	
	    public int getId() {
	        return id;
	    }
	
	    public void setId(int id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    public int getStudentId() {
	        return studentId;
	    }
	
	    public void setStudentId(int studentId) {
	        this.studentId = studentId;
	    }
	
	    public String getSex() {
	        return sex;
	    }
	
	    public void setSex(String sex) {
	        this.sex = sex;
	    }
	}

#### b. 修改litepal.xml中的代码 ####
---
	<list>
        <mapping class="com.xiaweizi.studentsystem.Student"></mapping>
    </list>
>运行一下程序，然后你就可以在data/data/包名的文件下看到数据库已经创建了：

![数据库.PNG](http://upload-images.jianshu.io/upload_images/4043475-7d01854c495230ad.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>然后把他导出来，借用工具打开，然后就是下面界面

![表的创建.PNG](http://upload-images.jianshu.io/upload_images/4043475-3e4c37de6dcd5cdb.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 4. 增、删、改、查 #
### 1. 添加数据 ###

		Student student = new Student();
        student.setName(name);
        student.setSex(sex);
        student.setStudentId(Integer.parseInt(studentId));
        student.save();
效果如下：
![添加数据.PNG](http://upload-images.jianshu.io/upload_images/4043475-bf48d4c2d74d168e.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 删除数据 ###
	DataSupport.deleteAll(Student.class, "id = ?", id +"");
>一行代码搞定，只要调用`DataSupport.deleteAll()`即可，第一参数，是要删除哪张表的数据，后面则为约束条件，不难看懂。
### 3. 修改数据 ###
		Student student = new Student();
        student.setName(name);
        student.setSex(sex);
        student.setStudentId(Integer.parseInt(studentId));
        student.updateAll("id = ?", id+"");
>还是要new一个实例，然后要设置更新的数据，最后调用`updateAll()`方法执行更新操作。参数跟删除数据很像，也是约束条件，如果不传参数，则修改所有的数据。

>这里需要注意一点，如果想让数据恢复成默认值，**是不能直接设置默认值的。**
>
>比如，如果让学号为0，是不能 `student.setStudentId(0);` 这是错误的！！！那么如果想恢复成默认值该怎么办呢，LitePal提供了`setToDefault()` 方法。

  	Student student = new Student();
    student.setToDefault("studentId");
    student.updateAll();
### 4. 查询数据 ###
	mList = DataSupport.findAll(Student.class);
>一行代码搞定，直接就可以查询数据库中Student表的所有数据，返回这个对象的集合。
>借用工具可以查看我们的所有数据：
![所有数据.PNG](http://upload-images.jianshu.io/upload_images/4043475-436d22171cb8b0d0.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到此一个简单的学生管理系统已经结束了，主要目的就是快速入门LitePal！
