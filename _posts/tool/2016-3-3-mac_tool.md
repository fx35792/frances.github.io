---
layout: post
title: Mac 使用技巧积累
category: 工具
tags: [mac]
description: Mac日常使用技巧积累
---

### Mac显示隐藏文件 - 第一种
	 命令行输入以下命令然后刷新Finder即可

```java
	defaults write com.apple.finder AppleShowAllFiles -bool false/true

```

### Mac显示隐藏文件 - 第二种

```
	1、打开Automator应用(一个小机器人)，可以搜索control + Space;
	2、将以下代码copy并运行，并保存成Toggle Hidden Files;

		STATUS=`defaults read com.apple.finder AppleShowAllFiles`
	if [ $STATUS == YES ];
	then
	    defaults write com.apple.finder AppleShowAllFiles NO
	else
	    defaults write com.apple.finder AppleShowAllFiles YES
	fi
	killall Finder
	3、设置键盘快捷键，在左边选择服务，然后勾上“Toggle Hidden Files”，在它的右边双击鼠标，然后按下你想要设定成为的快捷键,我这里是Command+Shift+.（点）;
	4、测试一下，会关掉Finder然后在打开就可以看见隐藏文件了

```
	![hidefile]({{ site.url }}/assets/image/hidefile.png)

### 使用命令行英文发音

```java
	say hello

```

### 查看当前目录文件大小

```java
	du -sh *

	//结果
	4.0K	404.html
	4.0K	CNAME
	4.0K	LICENSE.txt
	4.0K	README.md
	4.0K	_config.yml
	 28K	_includes
	 12K	_layouts
	160K	_posts
	224K	assets
	4.0K	ie.html
	4.0K	index.md
	4.0K	pages
	4.0K	sitemap.txt

```

### 截图方法

```
	shift + commmand + 3  //全屏幕
	shift + commmand + 4  //自由选取
	//图片保存在桌面
```

### 批量复制文件

```
	//进入目录，执行以下命令
	cp *.png *.gif /xxx		  //复制
	mv *.png *.gif /xxx      //剪切
```








