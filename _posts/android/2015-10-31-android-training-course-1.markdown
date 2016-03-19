---
layout: post
title: Android-Training-Course-Note1
category: android
keywords: Android
---

从今天起跟着Google官方Android Training课程进行学习，[官方地址](http://developer.android.com/intl/zh-cn/training/index.html)，并且记录在此，也是对以前知识的一个巩固和复习;

>参考了[http://hukai.me/](http://hukai.me/)中的Android-Training-Course翻译文章，特此感谢;

> 今天的这篇内容主要有：如何新建Android项目、添加ActionBar、兼容不同的设备、管理Activity的生命周期、使用Fragment建立官方动态的UI、数据保存、与其他应用进行交互等；不会对每个知识点进行详细说明了，记录下可能在开发中需要了解和注意的点就够了，用到的时候google下或者查查文档就会了;

### 1、新建Android项目

* 了解Android项目结构，清楚配置文件作用，例如：layout、res等;
* 注意理解配置文件含义、使用命令来编译程序;
* 使用Android Studio，个人已经渐渐的转到AS了;
* 尝试自己去布局xml，了解各个标签的使用;

### 2、兼容不同的设备

* 兼容主要有屏幕大小(layout)、语言(String)、图片(bitmap)、位置(dip,sp)等;
* 根据不同得区域设置不同得语言目录和字符串文件，[Android各国语言Values文件夹命名规则](http://my.oschina.net/quttap/blog/204499)，随系统切换设置自动读取;
* 使用dip(控件大小)、sp(文字大小)不推荐使用px;
* 适配不同的屏幕大小，使用layout-<screen_size>形式，例如:适配大屏幕的layout，layout-large/; 在此文件夹下放置布局文件，需要和其他屏幕的尺寸文件名一样，否则会出错;
* 根据不同的屏幕分辨率创建不同分辨率的图片资源目录;
* 注nodpi它可用于您不希望缩放以匹配设备密度的位图资源; .9图片资源,相关目录含义查看[官方文档](http://developer.android.com/intl/zh-cn/guide/topics/resources/providing-resources.html#AlternativeResources)，这里在开发时要用到
* 兼容低版本,AndroidManifest.xml指定最小和目标API级别，运行时动态去检查系统版本，根据版本高低使用不同平台风格和主题
例如:（Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB）;

### 3、管理Activity的生命周期

* 熟悉掌握Activity的[生命周期图](http://developer.android.com/training/basics/activity-lifecycle/starting.html);
* 必须掌握Activity的各个生命周期的使用时机，以及适合处理什么操作:例如：onStop()处理数据DB相关等;
* 生命周期成对出现，例如:onStart()和onStop()初始化和释放一一对应;
* UI主线程不要做过多的耗时操作，只用来操作UI现实，耗时操作使用异步去做，各个生命周期都是主线程所以可以操作UI，需要在子线程操作UI的需要使用Handler或其他方法post到UI主线程进行操作;
* 对使用Bitmap的地方进行手动使用bitmap，避免OOM、ANR;
* 谨慎使用Activity的Context和this,放置因为有引用无法释放Activity;
* onRestoreInstanceState()和onSaveInstanceState()用来恢复之前状态和保存一些系统不能保存得数据，比如：旋转屏幕得时候，生命周期会重新来过，需要保存一些以前状态得数据；使用onRestoreInstanceState()时不用检查Bundle是否null，因为系统仅在数据存在才会调用onRestoreInstanceState()方法;
* 注意：在onSaveInstanceState()中super.onSaveInstanceState(savedInstanceState)需要在方法得最后一行调用;

### 4、使用Fragment建立动态UI

* 前提要熟悉Fragment的生命周期，并且了解Activity和Fragment生命周期运行时机;
* 当新建一个Fragment时，必须重写onCreateView()方法来返回view布局，也是为一个需要回调得方法，其余得方法在需要时重写
* 添加Fragment有xml、代码两种方式添加，在开发中一般都是动态添加、移除、替换等操作;
* 导入V4包，在Activity中需要调用 getSupportFragmentManager()方法获取FragmentManager 对象，然后调用 beginTransaction() 方法创建一个FragmentTransaction对象，然后调用add()方法添加一个fragment;
* 通常fragment之间可能会需要交互，比如基于用户事件改变fragment的内容。所有fragment之间的交互需要通过他们关联的activity，两个fragment之间不应该直接交互;
* 通常都会定义一套固定的接口，实现类来负责几个fragment之间或者和activity之间的数据交互等，开发会很使用，UI也灵活;

### 5、数据保存 

* 三种形式，分别保存到Preferencexml文件、本地文件、数据库，根据自己得需求选择不同得方式来进行数据保存;
* Preference:使用最多得为SharedPreferences，可以进行简单得key－value; 键值对形式保存，存储方便适合保存简单数据结构，例如：引导图等;
* 本地文件:基本上都是读写本地存储文件，本地存储分内部和外部，外部需要读写权限，实际开发中使用过将一个序列化得对象存储在本地，使用时进行反序列化;
* Sqlite数据库：相关数据库增删改查，有相应得第三发包使用很方便[ORMLite](https://github.com/j256/ormlite-android)
* 例如:开发中使用存储来实现图片的三级缓存等;

### 其他

* 不要在非UI线程中初始化ViewStub，否则会返回null；
* 不要通过Bundle传递大块的数据，避免TransactionTooLargeException异常；
* 使用时才去创建对象，避免过多使用内存;
