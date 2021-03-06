---
layout: post
title: Fork本地项目和Github原始线上同步
category: 工具
keywords: Android,github,fork,git
---

保持Github Fork本地项目和Github原始地址线上同步

保持fork到本地的项目和线上的原始地址代码同步是一个经常会遇到的问题，本人开始的时候也不会，后来自己操作一遍后就
觉得很简单了，主要是理解merge的过程，其余就是敲几条命令的事;

### 三个步骤

1、添加原始作者的项目url地址,拉取项目,

{% highlight java%}
//你操作的时候Android-Tips要替换成你命名的名称

git remote -v //查看远程地址
Android-Tips	https://github.com/tangqi92/Android-Tips.git (fetch)
Android-Tips	https://github.com/tangqi92/Android-Tips.git (push)
origin	https://github.com/whiskeyfei/Android-Tips.git (fetch)
origin	https://github.com/whiskeyfei/Android-Tips.git (push)
git fetch Android-Tips //拉取原始项目

{% endhighlight %}

3、切换分支并且合并

{% highlight java%}

git checkout master //切换到master分支

{% endhighlight %}

4、手动解决冲突并提交

{% highlight java%}
git merge Android-Tips/master //手动merge
//手动解除冲突
git add xx
git commit -m "xxx" 
git push origin master
{% endhighlight %}

### 其他：
重命名远程仓库

{% highlight java%}
$: git remote rename github gh 
$: git remote 
origin gh

{% endhighlight %}

删除远程仓库

{% highlight java%}
$: git remote rm github
$: git remote origin
{% endhighlight %}

查看所有分支

{% highlight java%}
	git branch -a
{% endhighlight %}
