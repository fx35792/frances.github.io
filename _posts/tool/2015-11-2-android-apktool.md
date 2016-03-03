---
layout: post
title: Android Mac环境下反编译APK－Apktool
category: 工具
tags: [Android,apktool,jd-gui]
description: Android常用ADB命令,批量处理
---

## 一、软件准备

1、亲测[下载地址](http://pan.baidu.com/s/1hqvvHLM)

## 二、具体操作

1、将下载的软件放置适当目录下，本例子以 “/Users/xxx/Development/apktool”为例;<br/>
2、通过终端命令进入以上目录下，将要反编译的apk放置相同目录下;<br/>
3、执行命令,看到以下信息说明你成功了;<br/>

```java
apktool d xxx
I: Using Apktool 2.0.0-RC4 on ten.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /Users/xxx/Library/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
W: Cant find 9patch chunk in file: "drawable-hdpi-v4/water_mask_fg.9.png". Renaming it to *.png.
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```

4、以上可以看到apk中的xml信息和图片资源等;<br/>
5、开始反编译java源文件;<br/>
6、将刚才的apk文件后缀名称改成zip，然后解压得到classes.dex文件，将其复制到dex2jar-0.0.9.15目录下;<br/>
7、cd进入dex2jar-0.0.9.15目录，然后执行以下代码,执行成功后会生成一个classes_dex2jar.jar文件<br/>

```java
  sh dex2jar.sh classes.dex
```

8、最后，用jd-gui工具打开这个classes_dex2jar包就可以看到java源代码了<br/>

9、现在看过apk都做了保护，反编译只能编译到布局文件等资源，代码时看不到的，我现在的工具估计改更新了，如果有同学有更好的方法，也请回复给我哈，多谢！<br/>

















