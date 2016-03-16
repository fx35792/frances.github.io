---
layout: post
title: Android 内存泄漏工具使用
category: android
keywords: Android,memory
---

> 最近的一次事件让我对Android开发中内存泄漏重视起来，平时只忙着开发新的功能，往往会忽略掉内存，cpu等方面的使用情况，然而遇到ANR就要彻底找出内存泄漏并解决，由于Android设备规格不一
好一些的设备上不会出现问题，在一些低端的设备上就会出现各种问题，所以平时也要注意内存泄漏问题；

### 内存泄漏(memory leak)

A memory leak is a particular type of unintentional memory consumption by a computer program where the program fails to release memory when no longer needed. This condition is normally the result of a bug in a program that prevents it from freeing up memory that it no longer needs.This term has the potential to be confusing, since memory is not physically lost from the computer. Rather, memory is allocated to a program, and that program subsequently loses the ability to access it due to program logic flaws. 
看不懂 pass。。。

### DDMS HEAP工具

      Android DDMS 内存监测工具Heap，可以检测一个进程的内存变化，根据这个数值变化可以测试应用是否存在泄漏。
      用过debug肯定会用Heap了，连接设备，启动app，点debug右边的按钮即可（Update Heap 鼠标放上去能看到）
      开始GC，然后就操作app观察，Heap视图中部有一个data object，即数据对象，也就是我们的程序中对象
      正常情况下Total Size值都会稳定在一个范围内，反之如果则data object的Total Size值在每次GC后不会有明显的回落
      随着操作次数的增多Total Size的值会越来越大，就证明可能存在内存泄漏，然后通过其他工具来找出泄漏的地方；

### MAT
    
    使用DEAP初步判断页面存在内存泄露后，使用MAT具体分析出哪写对象没有释放，导致了内存没有释放；
    * 自行安装MAT工具；
    * 操作app，例如：进入某个页面，退出，点击Dump Hprof file按钮，等一会会打开MAT视图，没有安装会生成一个文件；
    * 先观察“Leak suspects”，找出比较大的问题
    * 通过Histogram查找引用链，例如：搜索 com.main.*;这样就能查找出这些包下面类引用情况
      自行右键获得信息，通过提示找出有问题的类或对象；
    * 然后就可以解决掉内存泄露问题了；

### 如何避免

    * 遵守生命周期，创建时创建，销毁时记得回收；
    * Bitmip和Drawable记得手动回收；
    * 静态对象引用Context，导致对象无法释放，从而导致Activity无法释放；
    * Handler回收；
    * 使用getApplicationContext();

### 参考文章

* [Android内存泄漏分析实战](http://my.oschina.net/u/2285044/blog/471027#OSC_h3_8)
* [【译】什么导致了Context泄露：Handler&内部类](http://www.cnblogs.com/kissazi2/p/4121852.html)
