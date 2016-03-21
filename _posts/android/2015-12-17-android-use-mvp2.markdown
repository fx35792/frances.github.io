---
layout: post
title: Android MVP 项目使用
category: android
keywords: Android,MVP
---

前段时间学习 MVP 思想，在原来 [SimpleNews](https://github.com/liuling07/SimpleNews) 的基础上，修改了数据层和 UI 层的逻辑，使用 RxJava + MVP 组合方式，重构完成 [SimpleNews.io](https://github.com/whiskeyfei/SimpleNews.io)，目前还存在很多需要优化的地方，今天说说 SimpleNews.io 中用到的 MVP 部分。

参考 [android-boilerplate](https://github.com/ribot/android-boilerplate), 添加 base Presenter 和 base View,以后所有的 Presenter 和 View 都需要继承 base ,这样在Presenter这一层才能使用统一方法回调接口，Model 暂时没有改变

### 好处

1、BasePresenter 将不同的view接口统一，不返回具体的view接口，通过方法操作，易于扩展;<br/>
2、view 接口的绑定统一化;

### 基础使用

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

### 项目使用       

首先看下我的项目结构:

![mvp]({{site.url}}/assets/image/mvp.png)

拿获取图片列表来举例，逻辑是这样：获取数据并解析结果给 view 展示。首先在 Fragment 中即 View 接口的实现类中
初始化我们的 Presenter 同时会初始化一个 Model，Fragment 调用 Presenter 中的方法，例如：loadImageList() 方法，接着去调用 Model中
的getImageList()方法,最后 Model 讲结果回调给 Presenter 层，Presenter 再回调给 View层，这样就完成了一个 MVP逻辑，以下是部分代码。
        
        //View
        private IImagePresenter mImagePresenter;

        @Override
        public void onCreate(@Nullable Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            mImagePresenter = new ImagePresenter(this);
        }

        //Presenter
        public class ImagePresenter extends BasePresenter<IImageView> implements IImagePresenter {
        private static final String TAG = "ImagePresenter";
        private IImageModel mImageModel;
        private Subscription mSubscription;

        public ImagePresenter(IImageView imageView) {
            attachView(imageView);
            mImageModel = new ImageModel();
        }

        @Override
        public void detachView() {
            super.detachView();
            if (mSubscription != null) {
                mSubscription.unsubscribe();
            }
        }

        @Override
        public void loadImageList() {
            checkViewAttached();
            mSubscription = mImageModel.getImageList()
                    .subscribeOn(Schedulers.io())
                    .doOnSubscribe(new Action0() {
                        @Override
                        public void call() {
                            getMvpBaseView().showProgress();
                        }
                    })
                    .observeOn(AndroidSchedulers.mainThread())
                    .subscribe(new Observer<List<ImageBean>>() {
                        @Override
                        public void onCompleted() {
                            getMvpBaseView().hideProgress();
                        }

                        @Override
                        public void onError(Throwable e) {
                            getMvpBaseView().hideProgress();
                            getMvpBaseView().showLoadFailMsg();
                        }

                        @Override
                        public void onNext(List<ImageBean> list) {
                            getMvpBaseView().onSuccess(list);
                        }
                    });
            }
        }

        //Model
        public class ImageModel implements IImageModel {

            private static final String TAG = "ImageModel";

            public Observable<List<ImageBean>> getImageList() {
                return Observable.just(Urls.IMAGES_URL).flatMap(new Func1<String, Observable<List<ImageBean>>>() {
                    @Override
                    public Observable<List<ImageBean>> call(final String url) {
                        //可以拼接url
                        return Observable.create(new Observable.OnSubscribe<List<ImageBean>>() {

                            @Override
                            public void call(final Subscriber<? super List<ImageBean>> subscriber) {
                                OkHttpUtils.ResultCallback<String> loadNewsCallback = new OkHttpUtils.ResultCallback<String>() {
                                    @Override
                                    public void onSuccess(String response) {
                                        Log.d(TAG, "onSuccess response:" + response);
                                        if(StringUtils.isEmpty(response)){
                                            subscriber.onError(null);
                                            return;
                                        }
                                        Gson gson = new Gson();
                                        List<ImageBean> iamgeBeanList = gson.fromJson(response, new TypeToken<List<ImageBean>>() {}.getType());
                                        if(ListUtils.isEmpty(iamgeBeanList)){
                                            subscriber.onError(null);
                                            return;
                                        }
                                        subscriber.onNext(iamgeBeanList);
                                        subscriber.onCompleted();
                                    }

                                    @Override
                                    public void onFailure(Exception e) {
                                        subscriber.onError(e);
                                    }
                                };
                                OkHttpUtils.get(url, loadNewsCallback);
                            }
                        });
                    }
                });
            }
        }

一个完整的MVP思想的使用就结束了，有没有觉得很简单，最初听名字以为是一个框架，学了才知道 MVP 是一种思想，开发多多使用，有问题欢迎留言，
也可以frok我的项目 [SimpleNews.io](https://github.com/whiskeyfei/SimpleNews.io) 一起学习。

### 参考文章

* [MVVM_Android-CleanArchitecture](http://rocko.xyz/2015/11/07/MVVM_Android-CleanArchitecture/)
* [Android中的MVP](http://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)
* [说说Android的MVP模式](http://toughcoder.net/blog/2015/11/29/understanding-android-mvp-pattern/)文章中有些话，摘自[说说Android的MVP模式](http://toughcoder.net/blog/2015/11/29/understanding-android-mvp-pattern/)一文
* [android-boilerplate](https://github.com/ribot/android-boilerplate)

### 推荐

* [WFAndroid_MVP](https://github.com/whiskeyfei/WFAndroidDemo/tree/master/WFAndroid_MVP) MVP Demo
* [SimpleNews.io](https://github.com/whiskeyfei/SimpleNews.io) 我的项目

