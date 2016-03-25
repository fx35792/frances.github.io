---
layout: post
title: String、StringBuffer、StringBuilder 纪录
category: 日常
keywords: Android,java,String
---

抽空总结了下 java 字符串相关知识点，平常使用最多的就是 String 类了，用于处理一些简单的字符串问题，比如：常量；当遇到相对复杂一些并且需要频繁拼接字符串时，就需要用到StringBuffer和StringBuilder了，以下都是自己平时的纪录方便查漏补缺，有些也摘自网上，这里做下整理，如果有侵犯您的内容，请通知我会及时修改。

## 大概情况：

String 字符串常量

StringBuffer 字符串变量（线程安全）

StringBuilder 字符串变量（非线程安全）


## 单个理解：

String：是不可改变的量，创建后不能在修改,多用于不经常改变的的常量，无频繁拼接操作的场景；


StringBuffer

	 * A modifiable {@link CharSequence sequence of characters} for use in creating

	 * strings, where all accesses are synchronized. This class has mostly been replaced

	 * by {@link StringBuilder} because this synchronization is rarely useful. This

	 * class is mainly used to interact with legacy APIs that expose it.


StringBuilder

	 * A modifiable {@link CharSequence sequence of characters} for use in creating

	 * strings. This class is intended as a direct replacement of

	 * {@link StringBuffer} for non-concurrent use; unlike {@code StringBuffer} this

	 * class is not synchronized.


看下两个的源码解释就很清楚了，两者都是可变字符串序列，基本用法和 String 一样，不同点就是 StringBuffer 是线程安全的，StringBuilder 是线程不安全的，其实就是同步和不同步的问题；


### 插播一条如何理解线程安全和线程不安全。。


比如一个 ArrayList 类，在添加一个元素的时候，它可能会有两步来完成：

1、在Items[Size] 的位置存放此元素；<br/>
2、增大 Size 的值；

在单线程运行的情况下，如果 Size = 0，添加一个元素后，此元素在位置 0，而且 Size=1；
而如果是在多线程情况下，比如有两个线程，线程 A 先将元素1存放在位置 0。但是此时 CPU 调度线程A暂停，线程 B 得到运行的机会。
线程B向此 ArrayList 添加元素2，因为此时 Size 仍然等于 0 （注意，我们假设的是添加一个元素是要两个步骤，而线程A仅仅完成了步骤1），
所以线程B也将元素存放在位置0。然后线程A和线程B都继续运行，都增加 Size 的值，结果Size都等于1。
那好，我们来看看 ArrayList 的情况，期望的元素应该有2个，而 实际元素是在0位置，造成丢失元素，故Size 等于 1。
这就是“线程不安全”了。
还让我联想到另外一个例子，就是银行存取钱过程，这里就不展开说了，大家想想对比一下实际情况就能明白。

赶紧扯回来String。。


## 基本用法


1、String

		String a = "abc";
		a.replace('a', 'd');
		System.out.println(a);//结果abc，不改变

2、StringBuffer

StringBuffer对象的初始化不像String类的初始化一样，Java提供的有特殊的语法，而通常情况下一般使用构造方法进行初始化

		StringBuffer sb = new StringBuffer("123");
		sb.append("123").append("123");

注：

在使用 StringBuffer 的时候，需要注意不能想用 String 它一样使用

		StringBuffer result = null;  
		结果警告：Null pointer access: The variable result can only be null at this location

3、StringBuilder
使用方式和 StringBuffer 一样，不多说了

还有一些基本操作字符串函数的方法，append、deleteCharAt、insert等等，这里就不一一介绍如何使用了，看名字就知道怎么用了。

注：

StringBuffer和String属于不同的类型，也不能直接进行强制类型转换，下面的代码都是错误的：

         StringBuffer s = “abc”;               //赋值类型不匹配
         StringBuffer s = (StringBuffer)”abc”;    //不存在继承关系，无法进行强转


StringBuffer对象和String对象之间的互转的代码如下：

		String s = “abc”;
		StringBuffer sb1 = new StringBuffer(“123”);
		StringBuffer sb2 = new StringBuffer(s);   //String转换为StringBuffer
		String s1 = sb1.toString();              //StringBuffer转换为String



## 性能对比

### String

String是不可变的对象, 因此每次对 String 类型进行改变的时候都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，会影响运行速度。

而 StringBuffer 类则结果就不一样，每次结果都会对 StringBuffer 对象本身进行操作，而不生成新的对象，再改变对象引用。
所以在一般情况下我们推荐使用 StringBuffer ，特别是字符串对象经常改变的情况下。而在某些特别情况下，
String 对象的字符串拼接其实是被 JVM 解释成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢，
而特别是以下的字符串对象生成中， String 效率是远要比 StringBuffer 快的：

		String S1 = “This is only a” + “ simple” + “ test”;
		StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”);
		你会很惊讶的发现，生成 String S1 对象的速度简直太快了，而这个时候 StringBuffer 居然速度上根本一点都不占优势。
		其实这是 JVM 的一个把戏，在 JVM 眼里，这个
		String S1 = “This is only a” + “ simple” + “test”; 其实就是：
		String S1 = “This is only a simple test”; 
		所以当然不需要太多的时间了。但大家这里要注意的是，如果你的字符串是来自另外的 String 对象的话，速度就没那么快了，譬如：
		String S2 = “This is only a”;
		String S3 = “ simple”;
		String S4 = “ test”;
		String S1 = S2 +S3 + S4;

		这时候 JVM 会规规矩矩的按照原来的方式去做
		当然这种简单的方式我们也不会去选择 StringBuffer 来拼接了，用 String 就好了。

### StringBuffer
在大部分情况下 StringBuffer > String

Java.lang.StringBuffer线程安全的可变字符序列。一个类似于 String 的字符串缓冲区，但不能修改。
虽然在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。

可将字符串缓冲区安全地用于多个线程。可以在必要时对这些方法进行同步，因此任意特定实例上的所有操作就好像是以串行顺序发生的，该顺序与所涉及的每个线程进行的方法调用顺序一致。
StringBuffer 上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。每个方法都能有效地将给定的数据转换成字符串，然后将该字符串的字符追加或插入到字符串缓冲区中。append 方法始终将这些字符添加到缓冲区的末端；而 insert 方法则在指定的点添加字符。

例如，如果 z 引用一个当前内容是“start”的字符串缓冲区对象，则此方法调用 z.append("le") 会使字符串缓冲区包含“startle”，而 z.insert(4, "le") 将更改字符串缓冲区，使之包含“starlet”。

### StringBuilder
在大部分情况下 StringBuilder > StringBuffer

java.lang.StringBuilder一个可变的字符序列是5.0新增的。此类提供一个与 StringBuffer 兼容的 API，但不保证同步。
该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。
如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快，两者的方法基本相同。

## 参考

* [浅谈 Java 字符串](http://github.thinkingbar.com/how-to-use-string/)
* [什么是线程安全和线程不安全](http://www.cnblogs.com/zwq194/archive/2012/06/26/2563567.html)
