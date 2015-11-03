---
layout: post
title: Android常用ADB命令
category: 工具
tags: [Android , adb]
description: Android常用ADB命令,批量处理
---

## 常用ADB命令

### 1、adb命令批处理

很多时候需要批量处理adb命令，例如：发送100条广播

* 方法一：使用adb shell
直接再命令行输入
```shell
  for((i=0;i<=100;i++))
  do
  echo $i ;
  adb shell am broadcast -a xxxx;
  i=$(($i+1));
  done
```
* 方法二：使用python脚本执行adb shell命令<br/>
   1、编写 test.py文件<br/>
```python
  import os
  
  n = 0
  while n < 100:
  print("send:",n)
  n = n + 1
  os.system("adb shell am broadcast -a xxxx")
```
  2、执行python test.py<br/>
  
  ```python
  root$ python loop.py
  ```
