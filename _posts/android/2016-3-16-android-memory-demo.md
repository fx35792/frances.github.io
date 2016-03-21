---
layout: post
title: Android 内存泄漏案例分析(Handler)
category: android
keywords: Android,memory
---

在Android开发开发中，操作不当很容易引起内存泄漏，这里主要记录下平时遇到问题，包括：静态变量、非静态的内部类、Handler，尤其是Handler
在开发中要格外的注意。

## 1、静态变量

{% highlight java %}
public class LeakActivityDemo extends Activity{
  private static TextView mTextView;
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mTextView = new TextView(this);
    mTextView.setText("demo");
  }
}
{% endhighlight %}

我这里这是一个例子，一般开发不会这么写，上面的代码在 Activity 结束的时候，mTextView 一直持有 this 引用，
导致整个 Activity 无法回收

解决:方法在 Activity 生命周期 onDestroy 时将 mTextView 置空，或者尽量少使用到静态变量。 
注意:在写代码时，要考虑到当前的变量是否持有当前 Activity 的引用，避免出现内存泄漏.

## 2、非静态内部类

{% highlight java %}
public class LeakActivity extends Activity{
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mHandler.postDelayed(new Runnable() {
      
      @Override
      public void run() {
        
      }
    }, 1);
  }
  
  //提示：This Handler class should be static or leaks might occur (com.mvp.view.LeakActivity.1)
  private Handler mHandler = new Handler() {
      @Override public void handleMessage(Message msg) {
          super.handleMessage(msg);
      }
  };
}
  {% endhighlight %}

在 Android 开发中，我们一般使用 Handler 处理异步操作，通常我们的代码会像上面一样实现，但是上面的代码可能存在泄漏，
其实编译器已经提示了警告，建议使用 static 静态标示。

原因：

1、首先在 java 中，非静态内部类会或者内部类会持有外部对象的引用，而静态内部类不会持有外部类的引用；

2、和 Handler 机制有关，我们知道 Handler 和 Looper 是一起工作的，在一个 Activity 起来的时候，会帮我们创建一个
在主线程中使用的 Looper，用来处理所有的消息，当 Hanlder 初始化好以后，就会有一个消息发送到了 Looper 的消息队列中，
而这个消息则包含了当前 Handler 的引用，这就是内存泄露的原因；

解决:

1、使用静态内部类，如果在内部类中使用了 Activiy 则要使用 WeakReference(弱引用)，并且需要注意判空。

2、还有Hanlder要在生命周期 ondestroy 时，取消该 Handler 对象的 Messag 和 Runnable。

例如：removeCallbacks(Runnable r)、removeMessages(int what)、mHandler.removeCallbacksAndMessages(null);  

{% highlight java %}
  private final MyHandler mHandler = new MyHandler(this);
  
  public static class MyHandler extends Handler {
      private final WeakReference<LeakActivity> mWeakActivity;
   
      public MyHandler(LeakActivity context) {
        mWeakActivity = new WeakReference<LeakActivity>(context);
      }
   
      @Override public void handleMessage(Message msg) {
          super.handleMessage(msg);
          if (mWeakActivity.get() != null) {
            mWeakActivity.get().todo();
      }
      }
  }
  
  public void todo(){
    //todo
  }
{% endhighlight %}

## 3、非静态内部类生成的静态变量

{% highlight java %}
private static MyClass myClass;
  private class MyClass {
    
  }

..

myClass = new MyClass();  
{% endhighlight %}  
这种是两者的综合体，只要是非静态内部类就会持有外部类的引用，如果外部类正好是 Activity ，那么会导致 Activity 无法回收；
处理方式和第一种很像，记得释放
### 参考文章

* [Android App 内存泄露之Handler](http://blog.csdn.net/zhuanglonghai/article/details/382330698)
* [【译】什么导致了Context泄露：Handler&内部类](http://www.cnblogs.com/kissazi2/p/4121852.html)
* [细话Java："失效"的private修饰符](http://droidyue.com/blog/2014/10/02/the-private-modifier-in-java/)
