---
layout: post
title: Android 内存泄漏demo
category: android
keywords: Android,memory
---

内存泄漏 (memory leak) Demo
 <p>这里这是一个例子，一般开发不会这么写上面的 demo 在 Activity 结束的时候，mTextView 一直持有 this 引用，导致Activity无法
回收，解决方法在 Activity 生命周期 onDestroy 时将 mTextView 置空，或者尽量少使用到静态变量。 
    所以在写代码时，要考虑到当前的变量是否持有当前 Activity 的引用，避免内存泄漏。</p>

### 1、静态变量

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

### 2、非静态内部类

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

Handler 提示可能存在泄漏，建议使用 static 静态标示
非静态内部类会引用外部对象，而静态内部类不会引用外部类对象，解决方法静态内部类使用WeakReference
需要注意每次都要判断空的情况。
还有Hanlder要在生命周期 ondestroy 时，取消该Handler对象的Message和Runnable
例如：removeCallbacks(Runnable r)、removeMessages(int what)、mHandler.removeCallbacksAndMessages(null);  

{% highlight java %}
  private MyHandler mHandler = new MyHandler(this);
  
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

### 非静态内部类生成的静态变量

{% highlight java %}
private static MyClass myClass;
  private class MyClass {
    
  }

..

myClass = new MyClass();  
{% endhighlight %}  
这种是两者的综合体，只要是非静态内部类就会持有外部类的引用，如果外部类正好是 Activity ，那么会导致 Activity 无法回收；

### 参考文章

* [Android App 内存泄露之Handler](http://blog.csdn.net/zhuanglonghai/article/details/382330698)
* [【译】什么导致了Context泄露：Handler&内部类](http://www.cnblogs.com/kissazi2/p/4121852.html)
