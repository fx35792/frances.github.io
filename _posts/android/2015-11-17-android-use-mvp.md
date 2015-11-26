---
layout: post
title: Android MVP简单使用
category: android
keywords: Android,MVP
---

> 前段时间接收了一个模块，看着代码让人捉急，于是将模块用MVP思想来优化了，考虑到不能太多的影响现有模块功能和其他调用方式，没有做太大的改动，只是将流程进行了梳理和优化，在以后新的模块中可以使用MVVM思想，比MVP优点更多，直接上案例； 

> MVP是一套理论概念，而不是一种具体实现,所以,找到能解决问题合适框架思想并加以运用是最重要的

> 最后送上一句话,再看资料时看到的，“Architecture is About Intent, not Frameworks” - Robert C. Martin (Uncle Bob)

### 介绍

* M-Model-模型、V-View-视图、P-Presenter-表示器,具体对比mvc、mvvm等自己Google;

### 注意

* 不能为了接口而抽象,适当的抽象;
* 考虑业务是否符合MVP;
* View过多时,不建议使用MVP,view增多导致Presenter增多,从而不以维护; 

### 具体使用

定义View接口

```java
  public interface IMainView {
    /**
     * view 显示文字
     * @param text
     */
    void showTextView(String text);
    }
```

定义Presenter接口

```java
  public interface IMainPresenter {
    /**
     * 显示文字事件分发
     */
    void show();
}
```

定义Model接口

```java
  public interface IMainModel {
    /**
     * 显示文字逻辑实现
     */
    void show();
}
```

定义结果回调接口，再Presenter中使用，给Model回调，然后再回调view，修改UI

```java
  public interface IMainCallback {
    /**
     * 显示结果文字 
     * @param string
     */
    void show(String string);
}
```

View实现

```java
  public class MainActivity extends Activity implements IMainView {

    private TextView mTextView;
    private Button mButton;
    private IMainPresenter mMainPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();
        mMainPresenter = new MainPresenter(this);
        mButton.setOnClickListener(new OnClickListener() {
            
            @Override
            public void onClick(View v) {
                mMainPresenter.show();
            }
        });
    }

    void initView() {
        mTextView = (TextView) findViewById(R.id.text);
        mButton = (Button) findViewById(R.id.loadButton);
    }

    @Override
    public void showTextView(final String text) {
        runOnUiThread(new Runnable() {
            
            @Override
            public void run() {
                mTextView.setText(text);
            }
        });
    }
    
}
```

Presenter实现

```java
public class MainPresenter implements IMainPresenter,IMainCallback{
    private IMainView mMainView;
    private IMainModel mUserModel;

    public MainPresenter(IMainView view) {
        mMainView = view;
        mUserModel = new MainModel(this);
    }
    
    @Override
    public void show() {
        mUserModel.show();
    }

    @Override
    public void show(String string) {
        mMainView.showTextView(string);
    }
}
```

Model实现

```java
public class MainModel implements IMainModel {
    IMainCallback call;

    public MainModel(IMainCallback call) {
        this.call = call;
    }

    @Override
    public void show() {
        call.show("123");
    }

}    
```

### 推荐文章

* [MVVM_Android-CleanArchitecture](http://rocko.xyz/2015/11/07/MVVM_Android-CleanArchitecture/)
* [Android中的MVP](http://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)

### Demo

[MVPSample-master](https://github.com/whiskeyfei/WFAndroidDemo/tree/master/MVPSample-master)

