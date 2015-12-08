---
layout: post
title: Android ViewStub、include使用
category: android
keywords: Android,ViewStub,include
---
# 一、include

> include用途：引入重复布局，使布局得到复用

### 1、使用

```
  <include
        android:id="@+id/top_layout_id"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        layout="@layout/title_top_layout" />

    /**
     * 直接通过this查找，因为include已经转换成layout中的布局
     */
    private void methodTwo() {
        TextView textView = (TextView)findViewById(R.id.textview);
        textView.setText("Top Title Two");
    }
    
    /**
     *  通过mView查找子view控件,mView为layout布局
     */
    private void methodOne() {
        ImageView imageView = (ImageView) mView.findViewById(R.id.imageview);
        TextView textView = (TextView) mView.findViewById(R.id.textview);
        textView.setText("Top Title One");
    }
```

### 2、注意事项

* 注意标签是是layout,和viewStub区分;
* 如果include设置了Id则需要通过Id来查找目标布局的根元素;

***

# 二、ViewStub

> ViewStub用途:布局优化，开发中经常听说view优化，但是真正用到得ViewStub可能不多，做下记录，今天优化代码将ViewStub加入,include标签就不用说了,开发经常用的标签，简单的包含父布局； 

### 1、使用场景

* 延迟加载或者很少几率才用到得view布局，比如:错误信息显示、占位等;
* 遇到重复的view布局，通过android:layout引入;
* 存在两者取其一等类似情况，没必要都初始化好;

### 2、注意事项

* <font color=#4094c7>inflate()被调用之后，返回的是父布局控件对象,不能再次调用inflate();</font>
* <font color=#4094c7>注意标签是android:layout不是layout,和include区分;</font>
* <font color=#4094c7>可以使用inflatedId重写ViewStub的父布局控件的Id,也就是说,如果ViewStup设置了inflatedId则需要通过inflatedId来查找目标布局的根元素;</font>

ViewStub源码:

```java
    public View inflate() {
        final ViewParent viewParent = getParent();

        if (viewParent != null && viewParent instanceof ViewGroup) {
            if (mLayoutResource != 0) {
                final ViewGroup parent = (ViewGroup) viewParent;
                final LayoutInflater factory = LayoutInflater.from(mContext);
                final View view = factory.inflate(mLayoutResource, parent,
                        false);
                //NO_ID＝－1即没有获取到mInflatedId
                if (mInflatedId != NO_ID) {
                    view.setId(mInflatedId);
                    //如果ViewStub的inflatedId不是NO_ID（－1）则把inflatedId设置为目标布局根元素的id,也就是layout根布局的id
                }

                final int index = parent.indexOfChild(this);
                parent.removeViewInLayout(this);
                final ViewGroup.LayoutParams layoutParams = getLayoutParams();
                if (layoutParams != null) {
                    parent.addView(view, index, layoutParams);
                } else {
                    parent.addView(view, index);
                }

                mInflatedViewRef = new WeakReference<View>(view);

                if (mInflateListener != null) {
                    mInflateListener.onInflate(this, view);
                }

                return view;
            } else {
                throw new IllegalArgumentException("ViewStub must have a valid layoutResource");
            }
        } else {
            throw new IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");
        }
    }

```

### 3、使用

```
     <ViewStub android:id="@+id/stub"
        android:inflatedId="@+id/subTree"
        android:layout="@layout/mySubTree"
        android:layout_width="120dip"
        android:layout_height="40dip" />

     ViewStub stub = (ViewStub) findViewById(R.id.stub);
     View view = stub.inflate();//这里的view就是include进来的view

     //这里的设置了inflatedId则通过inflatedId来获取子view

```
# 三、Demo

代码是最好的老师，[点我点我](https://github.com/whiskeyfei/WFAndroidDemo/tree/master/WPLayoutImp)

# 四、总结

官方说明ViewStub本质就是一个高度为0个View，默认不可见，只有通过调用setVisibility函数或者Inflate函数才会将其要装载的目标布局给加载出来,从而达到延迟加载效果，不需要的时候也可以不加载

# 五、推荐文章

* [Google ViewStub reference](http://developer.android.com/intl/zh-cn/reference/android/view/ViewStub.html)
* [ViewStub的应用](http://blog.csdn.net/hitlion2008/article/details/6737537)
* [使用惰性控件ViewStub实现布局动态加载](http://www.cnblogs.com/menlsh/archive/2013/03/17/2965217.html)
* [Android布局优化之ViewStub、include、merge使用与源码分析](http://blog.csdn.net/bboyfeiyu/article/details/45869393#t2)

