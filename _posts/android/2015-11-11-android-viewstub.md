---
layout: post
title: Android ViewStub使用
category: android
keywords: Android,ViewStub
---

> 用途:布局优化，开发中经常听说view优化，但是真正用到得ViewStub可能不多，做下记录，今天优化代码将ViewStub加入,include标签就不用说了,开发经常用的标签，简单的包含父布局； 

### 使用场景

* 很少几率才用到得view布局，比如:错误信息显示、占位等;
* 遇到重复的view布局，通过android:layout引入;
* 存在两者取其一等类似情况，没必要都初始化好;

### 注意事项

* inflate()被调用之后，返回的是父布局控件对象,不能再次调用inflate();
* 注意标签是android:layout不是layout,和include区分;
* 可以使用inflatedId重写ViewStub的父布局控件的Id;

### 使用

```
     <ViewStub android:id="@+id/stub"
        android:inflatedId="@+id/subTree"
        android:layout="@layout/mySubTree"
        android:layout_width="120dip"
        android:layout_height="40dip" />

     ViewStub stub = (ViewStub) findViewById(R.id.stub);
     View inflated = stub.inflate();
```

### 推荐文章

* [Google ViewStub reference](http://developer.android.com/intl/zh-cn/reference/android/view/ViewStub.html)
* [ViewStub的应用](http://blog.csdn.net/hitlion2008/article/details/6737537)
* [使用惰性控件ViewStub实现布局动态加载](http://www.cnblogs.com/menlsh/archive/2013/03/17/2965217.html)

