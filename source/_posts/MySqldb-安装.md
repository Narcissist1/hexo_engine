---
title: MySqldb 安装
date: 2016-04-13 18:46:13
tags: 
	- MySqldb
	- 小问题
categories: 技术相关
---

	tar zxvf MySQL-python-1.2.3.tar.gz
	cd MySQL-python-1.2.3
	python setup.py install


安装python-mysql

	pip install mysql-python

<!--more-->
修改某个链接
{% qnimg mysqldb_install.png %}

code:

	ls -l /usr/lib/mysql | grep libmysqlclient.so
	ln /usr/lib/mysql/libmysqlclient.so.16.0.0 /usr/lib/libmysqlclient.so.18


DEBUG笔记：
（终极解决：把/www/wdlinux/mysql/lib中的全部打包到/usr/lib/mysql和/usr/lib64/mysql中，然后重启）
先看看/usr/lib/mysql/中是否有so.16文件，没有的话从/usr/lib中或/www/wdlinux/mysql/lib中cp过来
查看so文件的链接信息：
ldconfig -v | grep mysql
正确时为：
{% qnimg mysqldb_install2.png %}

 
