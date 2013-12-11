---
layout: post
title: Linux下Django部署环境的搭建
date: 2013-12-11 16:19:00
description: Linux环境下对Python版本的升级和apache、Mod wsgi的安装
category: likeit
tags: [Django]
---

# 目标
搭建起apache部署Django项目的环境，操作系统CentOS 5，软件版本如下：

* python 2.7
* Django 1.5
* mod\_wsgi 3.3
* apache 2.2

# Python更新

## 安装
系统用的是python版本是2.6+，现在需要将python版本更新至2.7+版本了。

下载python2.7的源码包，然后编译安装，命令如下：

	./configure --enable-shared
	make all
	make install

这边需要注意的是，`configure`的参数`--enable-shared`，使用这个参数为python生成动态链接库。如果没有加此参数，编译安装也可以正常完成，不会影响python的使用。但是到安装mod\_wsgi会出现错误，错误内容为：

/usr/local/lib/python2.7/config/libpython2.7.a:  could not read symbols: Bad value  collect2ld returned 1 exit status apxs:Error: Command failed with rc=65536

这个错误就是在安装python时候没有`./configure --enable-shared`

## 检查版本
完成后一般的路径是 /usr/local/lib/python2.7，可以使用命令检查其版本是否正确

	/usr/loacl/bin/python2.7 -V

若是出现错误：python: error while loading shared  libraries: libpython2.7.so    .1.0: cannot open shared object file: No such file or directory。使用vim新建文件`/etc/ld.so.conf.d/python2.7.conf`，加入内容：`/usr/local/lib`。保存退出后运行：`ldconfig`，再次运行，看是否成功。

## 建立软链接
用于替换shell下的默认python命令的版本

	mv /usr/bin/python /usr/bin/oldpython
	ln -s /usr/local/bin/python2.7 /usr/bin/python

完成后可以输入命令`python -V`来检查时候成功，这里重命名旧版本python是为了让yum正常工作。centOS 5的yum使用的是python2.6版本，当更新到2.7后yum无法正常使用，通过修改/usr/bin/yum文件修正问题。在/usr/bin/yum文件的头部，把`#!/usr/bin/python`改为`#!/usr/bin/oldpython`，保存退出后正常使用yum。

# 安装apache服务器
可以使用yum来安装

	yum install -y httpd httpd-devel

其中httpd-devel是apache的apxs扩展，使用mod\_wsgi需要

# 安装mod\_wsgi
确保已经安装httpd-devel，如果已经安装了apache，但未安装httpd-devel，直接安装即可：

	yum install httpd-devel

mod\_wsgi同样采用源码编译安装，命令：

	./configure --with-python=/usr/local/bin/python2.7
	make all
	make install

这边要是有缺点啥就--with-啥。编译安装需要点时间。

安装完成后在apache的配置文件中（一般路径：/etc/httpd/conf/httpd.conf）加入

	LoadModule wsgi_module modules/mod_wsgi.so

# 其他
剩下的工作就是安装Django及其他的python库了。

刚开始安装遇到最多的问题就是python安装没有生成动态链接库，其他安装在网上都有好多资料，基本不会出错。接下来要学习下nginx部署Django项目。
