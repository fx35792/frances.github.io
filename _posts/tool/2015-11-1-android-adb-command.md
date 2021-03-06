---
layout: post
title: Android常用ADB命令
category: 工具
tags: [Android , adb]
description: Android常用ADB命令,批量处理
---

Android ADB 常用命令

### 1、启动和关闭adb服务
 

    adb kill-server  //关闭

  
    adb start-server //启动

### 2、adb shell绑定hosts

    //绑定
    adb shell
    root@android:/ # cat etc/hosts
    127.0.0.1       localhost
    root@android:/ # adb remount
    * daemon not running. starting it now on port 5038 *
    * daemon started successfully *
    remount succeeded
    root@android:/ # echo ip domain > etc/hosts
    root@android:/ # reboot
    e.g: echo 192.168.1.101 xxx.com > etc/hosts
         echo 192.168.1.102 yyy.com >> etc/hosts

### 3、adb通过ip链接设备


    //连接设备
    adb connect xxx.xxx.xxx.xxx
    e.g: adb connect 192.168.1.101

    //断开连接设备
    adb disconnect xxx.xxx.xxx.xxx
    e.g: adb disconnect 192.168.1.101


    //断开所有设备
    adb disconnect
    e.g: adb disconnect


### 4、adb发送广播

    adb shell am broadcast -a xxx

### 5、adb命令启动Activity

    //启动设置页面
    adb shell am start -n com.android.settings/.Settings

### 6、adb显示链接设备

    adb devices

    e.g: bogon:i$ adb devices
         List of devices attached 
         192.168.1.102:5555  device

### 7、adb命令查看内存使用情况


    //adb 查看内存信息
    adb shell dumpsys meminfo + packageName
    e.g: adb shell dumpsys meminfo com.dangbeimarket

### 8、adb安装和卸载应用


    adb install + apkpath //安装
    e.g: adb install D:/a.apk

    adb uninstall + packagename  //卸载
    e.g: adb uninstall com.dangbeimarket


### 9、adb命令批处理

很多时候需要批量处理adb命令，例如：发送100条广播

方法一：使用adb shell
直接再命令行输入

    for((i=0;i<=100;i++))
    do
    echo $i ;
    adb shell am broadcast -a xxxx;
    i=$(($i+1));
    done


方法二：使用python脚本执行adb shell命令<br/>
   1、编写 test.py文件<br/>

    import os
    
    n = 0
    while n < 100:
    print("send:",n)
    n = n + 1
    os.system("adb shell am broadcast -a xxxx")

    2、执行python test.py<br/>
    
    root$ python loop.py

### 10、查看cpu信息

    root@android:/ # cd proc
    root@android:/proc # cat cpuinfo                                               
    Processor : ARMv7 Processor rev 0 (v7l)
    processor : 0
    BogoMIPS  : 670.23

    processor : 1
    BogoMIPS  : 670.23

    Features  : swp half thumb fastmult vfp edsp neon vfpv3 
    CPU implementer : 0x41
    CPU architecture: 7
    CPU variant : 0x3
    CPU part  : 0xc09
    CPU revision  : 0

    Hardware  : Amlogic Meson6 g02 customer platform
    Revision  : 0020
    Serial    : 000000000000000c
    root@android:/proc #


    adb shell cat etc/proc/meminfo  //内存系统信息 

### 11、adb 截取日志log

    使用eclipse log时有缓冲区，所以不太好用，以下主要说下我平时用的

    adb logcat -c 清空缓存log信息;
    adb logcat -v thread > 1.log  保存到文件中
    adb logcat -v time
    adb logcat -s System.out  将System.out过滤出来


### 12、adb pull /push 使用

  使用eclipse ddms有时候不能导出文件，下面使用adb命令导入和导出文件

    adb remount //先要remount一下，后面可能会操作文件失败
    > remount succeeded

    //adb pull Test

    adb pull xxx xxx;//xxx为路径
    adb pull /data/data/com.mvp/files/text.txt data/     //data目录必须创建好

    //adb push Test

    adb push tex2.txt /data/data/com.mvp/files/


### 参考文章

* [ adb logcat 命令行用法](http://www.hanshuliang.com/?post=32)