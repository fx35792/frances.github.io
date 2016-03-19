---
layout: post
title: Android MVP简单使用2
category: android
keywords: Android,MVP
---


参考[android-boilerplate](https://github.com/ribot/android-boilerplate),添加base Presenter和base View,以后所有的P和V都需要继承base
这样在Presenter这一层才能使用统一方法回调接口，Model暂时没有改变

### 好处
1、BasePresenter将不同的view接口统一，不返回具体的view接口，通过方法操作，易于扩展;<br/>
2、view接口的绑定统一化;<br/>

### 具体使用

定义View接口，暂时为空，子类可以扩展添加

        public interface IMVPBaseView {

        }

定义Presenter接口


        public interface IMVPPresenter<V extends IMVPBaseView> {
            /**
             * 附加view回调接口
             * 
             * @param mvpBaseView
             */
            void attachView(V mvpBaseView);

             /**
             * 释放view接口
             */
            void detachView();

        }


BasePresenter实现

        public class BasePresenter<T extends IMVPBaseView> implements IMVPPresenter<T> {
            private T mMvpBaseView;

            @Override
            public void attachView(T mvpBaseView) {
                mMvpBaseView = mvpBaseView;
            }

            @Override
            public void detachView() {
                mMvpBaseView = null;
            }

            public T getMvpBaseView() {
                return mMvpBaseView;
            }

            /**
             * 是否附加view接口
             * 
             * @return true mvpBaseView != null
             */
            public boolean isViewAttached() {
                return mMvpBaseView != null;
            }

            /**
             * 检查是否绑定view接口
             */
            public void checkViewAttached() {
                if (!isViewAttached())
                    throw new RuntimeException(
                            "Please call BasePresenter.attachView(IMVPBaseView) before"
                                    + " requesting data to the Presenter");
            }
        }

### 参考文章

* [MVVM_Android-CleanArchitecture](http://rocko.xyz/2015/11/07/MVVM_Android-CleanArchitecture/)
* [Android中的MVP](http://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)
* [说说Android的MVP模式](http://toughcoder.net/blog/2015/11/29/understanding-android-mvp-pattern/)文章中有些话，摘自[说说Android的MVP模式](http://toughcoder.net/blog/2015/11/29/understanding-android-mvp-pattern/)一文
* [android-boilerplate](https://github.com/ribot/android-boilerplate)

### Demo

* [WFAndroid_MVP](https://github.com/whiskeyfei/WFAndroidDemo/tree/master/WFAndroid_MVP)

