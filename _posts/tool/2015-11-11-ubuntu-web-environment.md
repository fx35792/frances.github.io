---
layout: post
title: ubuntu下配置web开发环境搭建
category: 工具
tags: [ubuntu、mysqlphp、apache、phpmyadmin]
description: ubuntu环境下配置web开发环境
---
> 趁着1111在阿里云买了主机，从此开始了和远程主机得勾勾搭搭，趁着热乎先把web环境搭建好，标配LAMP(Linux+Apache+MySQL+PHP/Perl/Python),以下安装环境为ubuntu系统命令模式下进行;

## 一、Mysql

* sudo apt-get install mysql-server
* sudo apt-get update
* sudo service mysql restart

1、执行第一条命令，如果出现缺少软件包无法下载情况，则限执行sudo apt-get update命令，再执行第一条即可;<br/>
2、进入蓝色背景设置界面，设定密码神马的;<br/>
3、重启、登录;<br/>

```
mysql(sudo service mysql restart)

mysql -u root -p（需要输入刚才设定的密码）
```

4、php支持mysql<br/>

```
sudo apt-get install php5-mysql
```

## 二、Apache

* sudo apt-get install apache2 //安装
* sudo /etc/init.d/apache2 restart //重启

1、Apache安装完成后，默认的网站根目录是/var/www/html;<br/>
2、不出意外得话在浏览器中访问localhost或者127.0.0.1或者真实ip就可以打开apache得默认页面;<br/>
3、将网站得默认目录修改成www下，两个文件etc/apache2目录下的apache2.conf和etc/apache2/sites-available/000-default.conf文件;<br/>

```
//以下是部分命令，不完整
//通过cat命令查看文件
/etc/apache2# cat apache2.conf

<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

sudo vi etc/apache2/sites-available/000-default.conf
//修改目录
DocumentRoot /var/www

```

4、改完后将apache重启<br/>

```
sudo /etc/init.d/apache2 restart
```

5、apache支持mysql<br/>

```
sudo apt-get install libapache2-mod-auth-mysql
```

## 三、PHP

1、安装<br/>

```
sudo apt-get install php5
```

2、让Apache支持php<br/>

```
sudo apt-get install libapache2-mod-php5
```

3、安装php5-gd模块<br/>

```
sudo apt-get install php5-gd
```

4、测试<br/>

```
//编写info.php文件，写入"<?php phpinfo(); ?>"保存
http://localhost/info.php
```

## 四、phpmyadmin

1、安装<br/>

```
sudo apt-get install phpmyadmin
```

2、建立phpmyadmin和网站目录链接<br/>

```
cd /var/www
sudo ln -s /usr/share/phpmyadmin phpmyadmin
```

3、测试<br/>

```
http://localhost/phpmyadmin
```













