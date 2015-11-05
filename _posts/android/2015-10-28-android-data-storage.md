---
layout: post
title: Android-data-storage
category: android
keywords: Android,data,storage
---

## Android文件存储那些事

>总的来讲Android中数据存储不外乎以下几种，分别是：SharedPreferences、Storage、Sqlite数据库以及网络；
>以上几种方式各有各的优点，我们只需要再合适的项目时机选用合适的方式来存储就OK了，不要大材用;

## 1、Shared Preferences

* 键值集最适合保存配置信息，例如:背景设置、第一次启动引导图等；
* 只能保存基本数据类型以key/value的形式；
* 选择使用只是文件存储或者Activity的一个配置文件存储方式;
* 官方阅读[shared-preferences](http://developer.android.com/intl/zh-cn/training/basics/data-storage/shared-preferences.html);

写入

```java
//name为文件名称，key为存储的键
SharedPreferences sharedPref = this.getSharedPreferences(name, Context.MODE_PRIVATE);
SharedPreferences.Editor editor = sharedPref.edit();
editor.putInt(key, 1);
editor.commit();
```

读取

```java
SharedPreferences sharedPref = this.getSharedPreferences(name,Context.MODE_PRIVATE);
long value = sharedPref.getString(key, "");
```

## 2、保存文件

* 使用外部存储必须在manifest配置读写权限;
* 存储时检查是否有权限读写、空间是否够用，防止IO异常发生;
* 事先不知道大小的，可以在出现IO异常时捕获进行处理;
* 大文件要放到子线程操作;
* 当用户卸载app时，会把内部存储相关的文件都清除干净，而外部存储系统仅仅会删除根目录(getExternalFilesDir())下的相关文件;

```java
//配置权限
 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
 <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

//选择目录
getFilesDir()  //  /data/data/package/files/
getCacheDir()
getExternalFilesDir()
getExternalStoragePublicDirectory()
```

注意检查状态

```java
Environment.getExternalStorageState()
Environment.MEDIA_MOUNTED
Environment.MEDIA_MOUNTED_READ_ONLY
```

## 3、SQL数据库

* 第三发包使用很方便[ORMLite](https://github.com/j256/ormlite-android)
* 操作完成后记得关闭数据库链接

## 总结

* 简单数据使用SharedPreferences;
* 结构化使用Sqlite数据库；
* 重要文件注意要保密



