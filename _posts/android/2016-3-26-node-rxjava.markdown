---
layout: post
title: RxJava 初学和在 Android 项目中的简单应用
category: Android
keywords: Android,RxJava
---

前段时间关于 RxJava 的内容铺天盖地，作为一个 Android 开发，总有一种必须要学一学的感觉，今天来说说 RxJava，当然了只是简单应用，
自己也没有那么深入研究，不过可以在使用过程中慢慢去了解 RxJava 的机制。本文例子有一些来源于网上，也是一步一步跟着看下来，如有引用不当
请及时告知，我及时更改。


## 观察者模式

这里需要说一下观察者模式，RxJava 是扩展的观察者模式，先要了解下观察者模式，我在 java 基础中有纪录，可以去看[设计模式－观察者模式](../tech/2015-11-26-java-base.md)，自己敲一个 Demo 就行了，最常见的 Android 中Button绑定点击监听事件，其实不了解观察者模式也能理解这种点击监听事件处理，
觉得用的非常合理，我有一个 Button ，想点击时跳转另一个页面，就需要给 Button 绑定一个 Listener ，跳转的逻辑在监听里面做。
按照观察者模式思考的话，Button ＝ 被观察者，Listener 是观察者，二者绑定在一起，就形成了 Button 被点击，Listener 被执行。完事。


## RxJava 的观察者模式

RxJava 有四个基本概念：Observable (可观察者，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 onNext() （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：onCompleted() 和 onError()。

onCompleted(): 事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志。
onError(): 事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

以上完全摘自[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083#toc_1),写的特别好，自己看了不少20遍。

## 练习

刚开始看 [Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083#toc_1)的时候有些确实不太明白，然后自己又跟着[NotRxJava懒人专用指南](http://www.devtf.cn/?p=323) 敲了所有的代码，然后才渐渐的理解 RxJava 原来是想达到这样的效果，然后在开始练习 RxJava 相关的 case 就很容易理解了。

1、NotRxJava

这例子是跟着[NotRxJava懒人专用指南](http://www.devtf.cn/?p=323) 完成的例子，这里简单说一说。

有个需求是这样描述的；从网络获取 Cat 列表数据，根本某个条件找出相应的 Cat；
再没有用过 RxJava 时，我们可能就会一步一步的进行操作，先获取数据 -> 解析数据 -> 循环匹配。看似非常完美，然后某一天又增加了一天过滤条件，就要在原来的基础上
修改，然后。。然后。。。改。。改。。，我们肯定不想过这样的日子。


下面用 [NotRxJava懒人专用指南](http://www.devtf.cn/?p=323) 来简单实现这样一个功能，在使用时简化了很多，传入条件，直接返回结果，不需要了解内部的实现，符合最简。


调用方式

	  CatsHelper4 catsHelper4 = new CatsHelper4();
	        catsHelper4.saveTheCutestCat(Constants.BAIDU).start(new Callback<String>() {
	            @Override
	            public void onResult(String result) {
	                Log.e(TAG,"catsHelperTest4 result:"+result);
	            }

	            @Override
	            public void onError(Exception e) {
	                Log.e(TAG,"catsHelperTest4 e:"+e);
	            }
	        });

		public class CatsHelper4 {
		    ApiWrapper3 apiWrapper = new ApiWrapper3();

		    public AsyncJob<String> saveTheCutestCat(final String query) {
		       final AsyncJob<List<Cat>> catsListAsyncJob = apiWrapper.queryCats(query);
		//        这 16 行代码只有一行是对我们有用（对于逻辑来说）的操作：
		//        findCutest(result)

		        final AsyncJob<Cat> cutestCatAsyncJob = new AsyncJob<Cat>(){

		            @Override
		            public void start(final Callback<Cat> callback) {
		                catsListAsyncJob.start(new Callback<List<Cat>>() {
		                    @Override
		                    public void onResult(List<Cat> result) {
		                        callback.onResult(findCutest(result));
		                    }

		                    @Override
		                    public void onError(Exception e) {
		                        callback.onError(e);
		                    }
		                });
		            }
		        };

		        AsyncJob<String> stringAsyncJob = new AsyncJob<String>() {
		            @Override
		            public void start(final Callback<String> cutestCatCallback) {
		                cutestCatAsyncJob.start(new Callback<Cat>() {
		                    @Override
		                    public void onResult(Cat result) {
		                        apiWrapper.store(result).start(new Callback<String>() {
		                            @Override
		                            public void onResult(String result) {
		                                cutestCatCallback.onResult(result);
		                            }

		                            @Override
		                            public void onError(Exception e) {
		                                cutestCatCallback.onError(e);
		                            }
		                        });
		                    }

		                    @Override
		                    public void onError(Exception e) {
		                        cutestCatCallback.onError(e);
		                    }
		                });
		            }
		        };

		        return stringAsyncJob;
		    }

		    private Cat findCutest(List<Cat> cats) {
		        return cats.get(0);
		    }
		}	        

		public class ApiWrapper3 {
		    Api api = new ApiTest();

		    public AsyncJob<List<Cat>> queryCats(final String query) {
		        return new AsyncJob<List<Cat>>() {
		            @Override
		            public void start(final Callback<List<Cat>> catsCallback) {
		                api.queryCats(query, new Api.CatsQueryCallback() {

		                    @Override
		                    public void onCatListReceived(List<Cat> cats) {
		                        catsCallback.onResult(cats);
		                    }

		                    @Override
		                    public void onError(Exception e) {
		                        catsCallback.onError(e);
		                    }
		                });
		            }
		        };
		    }

		    public AsyncJob<String> store(final Cat cat) {
		        return new AsyncJob<String>() {
		            @Override
		            public void start(final Callback<String> callback) {
		                api.store(cat, new Api.StoreCallback() {
		                    @Override
		                    public void onCatStored(String s) {
		                        callback.onResult(s);
		                    }

		                    @Override
		                    public void onStoreFailed(Exception e) {
		                        callback.onError(e);
		                    }
		                });
		            }
		        };
		    }
		}

首先调用非常简单，生成一个对象 CatsHelper4 ，使用 saveTheCutestCat 方法，传入一个 url 地址，成功则返回 Cat result，失败则返回 Exception 信息；

saveTheCutestCat 方法会触发queryCats(query) 方法，并 new 一个 AsyncJob<Cat> 对象，queryCats 是一个异步操作，真正起获取网络请求的是在 ApiTest 中，它实现了Api接口，这部分先不考虑，这部分可以单独拿出来，我们就当数据已经获取了，并且赋值给了 catsListAsyncJob 变量，这时候 saveTheCutestCat 方法中返回一个 AsyncJob<String> 对象，其实就是结果，但是这个结果触发是在获取数据之后，由catsHelper4.saveTheCutestCat(Constants.BAIDU).start() ,start 方法开始，根据 url
stringAsyncJob.start -> cutestCatAsyncJob.start -> apiWrapper.store -> result，相当于是一个逆向操作，在写的时候是一步一步正向操作，而在调用的时候
需要直接拿结果，结果没有再向上获取上一个结果，形成一个闭环。

这个例子可能看上去太过于麻烦，当时看的时候自己也觉得是，但是当你真正写了一遍以后，就会发现，我们平时编程就应该有这种思想，使用方式特别简单，接口抽象等等，这样
才能做好易于扩展。

但说回来，这就是一个类似于 Rxjava 的例子，可能远远不够，因为没有观察者模式，所以下面我们来练习使用 RxJava 的基本方法；


2、Rxjava 练习

练习使用创建Observable、快速创建 create from just等、map flatMap filter doOnNext 等函数使用

* map: 事件对象的直接变换,非常灵活和实用，例子中有很多，string -> int;
* flatMap: 事件操作符
	flatMap() 和 map() 有一个相同点：它也是把传入的参数转化之后返回另一个对象，但需要注意，和 map() 不同的是， flatMap() 中返回的是个 Observable 对象，并且这个 Observable 对象并不是被直接发送到了 Subscriber 的回调方法中。
* filter:用于过滤等。
* take:获取个数。
* doOnNext:允许我们在每次输出一个元素之前做一些额外的事情	

	    //将字符串数组 names 中的所有字符串依次打印出来,并且支持不完整得回调

	    @Test
	    public void test1(){
	        String [] names ={"111","222","333"};
	        Observable.from(names).subscribe(new Action1<String>() {
	            @Override
	            public void call(String name) {
	                System.out.println("test1 name:"+name);
	            }
	        });
	    }

	    //just 将传入的参数依次打印
	    //    onNext:1
	    //    onNext:2
	    //    onNext:3
	    //    test2 onCompleted
	    @Test
	    public void test2(){
	        Observable.just("1","2","3").subscribe(new Observer<String>() {
	            @Override
	            public void onCompleted() {
	                System.out.println("test2 onCompleted");
	            }

	            @Override
	            public void onError(Throwable e) {
	                System.out.println("test2 e:"+e);
	            }

	            @Override
	            public void onNext(String s) {
	                System.out.println("onNext:"+s);
	            }
	        });
	    }


		 //将字符串数组 names 中的所有字符串依次打印出来
		    //    依次调用onNext("111")
		    //    依次调用onNext("222")
		    //    依次调用onNext("333")
		    @Test
		    public void test3(){
		        String [] names ={"111","222","333"};
		        Observable.from(names).subscribe(new Observer<String>() {
		            @Override
		            public void onCompleted() {
		                System.out.println("test3 onCompleted");
		            }

		            @Override
		            public void onError(Throwable e) {
		                System.out.println("test3 e:"+e);
		            }

		            @Override
		            public void onNext(String s) {
		                System.out.println("test3 onNext:"+s);
		            }
		        });
		    }


	    //将字符串数组 names 中的所有字符串依次打印出来
	    //    依次调用onNext("111")
	    //    依次调用onNext("222")
	    //    依次调用onNext("333")
	    @Test
	    public void test4(){
	        //1:被观察者
	        String [] names ={"111","222","333"};
	        Observable observable = Observable.from(names);

	        //2:观察者
	        Action1 onNextAction = new Action1<String>() {
	            @Override
	            public void call(String s) {
	                System.out.println("test4 call"+s);
	            }
	        };

	        Action1<Throwable> onErrorAction = new Action1<Throwable>() {
	            @Override
	            public void call(Throwable e) {
	                System.out.println("test4 call e:"+e);
	            }
	        };


	        Action0 onCompletedAction = new Action0() {
	            @Override
	            public void call() {
	                System.out.println("test4 call onCompletedAction");
	            }
	        };
	        //3:订阅:被观察者被观察者订阅
	        observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
	    }

		//循环输出list
	    //    test5 call :list:0
	    //    test5 call :list:1
	    //    test5 call :list:2
	    //    test5 call :list:3
	    //    test5 call :list:4
	    //    test5 call :list:5
	    //    test5 call :list:6
	    //    test5 call :list:7
	    //    test5 call :list:8
	    //    test5 call :list:9
	    @Test
	    public void test5(){
	        Observable.from(Data.getCats().get(0).getlist()).subscribe(new Action1<String>() {
	            @Override
	            public void call(String s) {
	                System.out.println("test5 call :"+s);
	            }
	        });
	    }	    


	    //cats -> cat.list -> s
	    //1:被观察者

	    //2:数据转换

	    //3:被观察者被观察者订阅

	    //4:观察者
	    @Test
	    public void test8(){
	        Observable.from(Data.getCats()).flatMap(new Func1<Cat, Observable<String>>() {
	            @Override
	            public Observable<String> call(Cat cat) {
	                return Observable.from(cat.getlist());
	            }
	        }).subscribe(new Action1<String>() {
	            @Override
	            public void call(String s) {
	                System.out.println("test8 call :"+s);
	            }
	        });
	    }


        //操作符 用来转换string

        Observable.just("hello World").map(new Func1<String, String>() {

            @Override
            public String call(String s) {
                return s + ":test";
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onCompleted() {
            }

            @Override
            public void onError(Throwable e) {
            }

            @Override
            public void onNext(String s) {
                System.out.println("onNext s:"+s);
            }
        });


        //string -> hashCode
        Observable.just("Hello, world!").map(new Func1<String, Integer>() {

            @Override
            public Integer call(String s) {
                return s.hashCode();
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onCompleted() {
            }

            @Override
            public void onError(Throwable e) {
            }

            @Override
            public void onNext(Integer integer) {
                System.out.println("onNext integer:"+integer);
            }
        });


        //转换三次 string -> int > string > int
        Observable.just("Hello, world!").map(new Func1<String, Integer>() {

            @Override
            public Integer call(String s) {
                return s.hashCode();
            }
        }).map(new Func1<Integer, String>() {

            @Override
            public String call(Integer integer) {
                return Integer.toString(integer);
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onCompleted() {
            }

            @Override
            public void onError(Throwable e) {
            }

            @Override
            public void onNext(String s) {
                System.out.println("onNext s:"+s);
            }
        });


            @Test
	    public void ObserverList(){
	        Observable.create(new Observable.OnSubscribe<List<String>>() {

	            @Override
	            public void call(Subscriber<? super List<String>> subscriber) {
	                List<String> list = new ArrayList<>();
	                for (int i = 0;i<10;i++){
	                    list.add(i + "--");
	                }
	                subscriber.onNext(list);
	            }
	        }).subscribe(new Observer<List<String>>() {
	            @Override
	            public void onCompleted() {

	            }

	            @Override
	            public void onError(Throwable e) {

	            }

	            @Override
	            public void onNext(List<String> strings) {
	                System.out.println("flatMapTest onNext strings:"+strings);
	            }
	        });
	    }

		// list -> string -> 不为null -> 获取5个
	    @Test
	    public void flatMapTest(){
	        //flatMap操作符
	        Observable.create(new Observable.OnSubscribe<List<String>>() {

	            @Override
	            public void call(Subscriber<? super List<String>> subscriber) {
	                List<String> list = new ArrayList<>();
	                for (int i = 0;i<10;i++){
	                    list.add(i + "--");
	                }
	                subscriber.onNext(list);
	            }
	        }).flatMap(new Func1<List<String>, Observable<String>>() {
	            @Override
	            public Observable<String> call(List<String> strings) {
	                return Observable.from(strings);
	            }
	        }).filter(new Func1<String, Boolean>() {
	            @Override
	            public Boolean call(String s) {
	                //滤掉null值
	                return s != null;
	            }
	            //take只需要5个 获取的意思
	        }).take(5).doOnNext(new Action1<String>() {
	            @Override
	            public void call(String s) {
	                //doOnNext()允许我们在每次输出一个元素之前做一些额外的事情
	            }
	        }).subscribe(new Observer<String>() {
	            @Override
	            public void onCompleted() {

	            }

	            @Override
	            public void onError(Throwable e) {

	            }

	            @Override
	            public void onNext(String s) {
	                System.out.println("flatMapTest onNext s:"+s);
	            }
	        });
	    }	


		@Test
	    public void subscriptionTest(){
	        Subscription subscription = Observable.just("").subscribe(new Action1<String>() {
	            @Override
	            public void call(String s) {

	            }
	        });
	        subscription.isUnsubscribed();//检查是否已经取消订阅
	        subscription.unsubscribe();//停止整个链
	    }   

## Android使用

未完待续－	    

## 推荐

* [WFRxJavaDemo](https://github.com/whiskeyfei/WFAndroidDemo/tree/master/WFRxJavaDemo) demo例子都在里面