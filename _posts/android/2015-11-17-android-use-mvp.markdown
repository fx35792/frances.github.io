---
layout: post
title: Android MVP简单使用
category: android
keywords: Android,MVP
---

前段时间接了一个模块，将模块用MVP思想来优化了，考虑到不能太多的影响现有模块功能和其他调用方式，没有做太大的改动，只是将流程进行了梳理和优化，在以后新的模块中可以使用MVVM思想，比MVP优点更多，直接上案例； 

MVP(Model-View-Presenter)是一套理论概念，而不是一种具体实现,需要注意MVP仅用于应用中的GUI部分，它并不是整个应用的架构方式，一个应用的主要的架构应该包括基础组件，业务逻辑层和GUI展示层，而MVP仅是用于展示层的设计模式。另外，它是一个方法论的东西，没有固定的实现方式，只要能体现出它的方法就可以算是MVP。

### MVC

简单说下MVC(Model-View-Controller),MVC在网站开发中用的多，以前用thinkphp时就是MVC，MVC机构分为三部分，实体层的Model，视图层的View，控制层的Controller。
View就是我们看到的程序界面，接受用户的操作，Model就是数据，Controller用于更新View和Model。
这样三部分都将自己的业务分开，不必干扰其他模块部分，和设计模式当中类功能的单一职责相似。

### MVP

MVP模式（Model-View-Presenter）应该是MVC在Android开发上的一种变化模式，其思想是一样的，都是将三层业务分离。
View对应Activity，业务逻辑Presenter，Model还是数据。
这样划分以后，三层工作逻辑很清晰，view只管相应用户等系统事件，具体操作全都丢到Persenter中，而view不能直接操作model，需要通过Presenter中转。

### 作用
    
* 分离了Activity中的UI和业务逻辑，降低了耦合程度，以后如果更换view层会很方便，因为UI层和业务层互不影响，只存在一个调用关系;
* 业务逻辑层代码可以复用，降低了维护成本;
* 增强了代码的可读性，一层一层调用会很清晰;
* 可以进行单元测试，通过后和UI层在进行合并，提高效率;

### 注意

* 不能为了接口而抽象,适当的抽象;
* 考虑业务是否符合MVP;
* View过多时,不建议使用MVP,view增多导致Presenter增多,从而不以维护，不过一般view也不会太多;

### 约束条件

* Model与View不能直接通信，只能通过Presenter;
* Model和View是接口，Presenter持有的是一个Model接口和一个View接口;
* Model和View都应该是被动的，一切都由Presenter来主导;
* Model应该把与业务逻辑层的交互封装掉，换句话说Presenter和View不应该知道业务逻辑层；
* View的逻辑应该尽可能的简单，不应该有状态。当事件发生时，调用Presenter来处理，并且不传参数，Presenter处理时再调用View的方法来获取;

### 具体使用

定义View接口

      public interface IMainView {
        /**
         * view 显示文字
         * @param text
         */
        void showTextView(String text);
        }


定义Presenter接口


          public interface IMainPresenter {
            /**
             * 显示文字事件分发
             */
            void show();
        }


定义Model接口


          public interface IMainModel {
            /**
             * 显示文字逻辑实现
             */
            void show();
        }


定义结果回调接口，再Presenter中使用，给Model回调，然后再回调view，修改UI


        public interface IMainCallback {
            /**
             * 显示结果文字 
             * @param string
             */
            void show(String string);
        }


View实现


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


Presenter实现


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


Model实现

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

### 参考文章

* [MVVM_Android-CleanArchitecture](http://rocko.xyz/2015/11/07/MVVM_Android-CleanArchitecture/)
* [Android中的MVP](http://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)
* [说说Android的MVP模式](http://toughcoder.net/blog/2015/11/29/understanding-android-mvp-pattern/)文章中有些话，摘自[说说Android的MVP模式](http://toughcoder.net/blog/2015/11/29/understanding-android-mvp-pattern/)一文

### Demo

* [WFAndroid_MVP](https://github.com/whiskeyfei/WFAndroidDemo/tree/master/WFAndroid_MVP)

