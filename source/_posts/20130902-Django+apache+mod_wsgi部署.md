---
layout: post
title: Django+apache+mod_wsgi部署
date: 2013-09-02 10:20:00
description: Django+apache+mod_wsgi部署
tags: [Django,Python]
urlname: django-apache-mod-wsgi-deployment
---

# 一、安装：

安装Apache、Python、Django、mod_wsgi、项目所需的依赖（数据库驱动、第三方库等）

# 二、项目设置：

1、把整个项目放到服务器上

2、改变DEBUG配置，修改`settings.py`文件的DEBUG属性

	DEBUG=False

3、数据库连接配置，修改`settings.py`的数据库连接配置

4、修改允许访问的HOST，修改`settings.p`y的`ALLOWED_HOSTS=['*']`

5、添加404和500页面，把页面文件放到template目录下

6、静态文件配置，修改`settings.py`的`STATIC_ROOT`的值，此属性的值是部署后静态文件保存的目录。

7、收集静态文件，运行命令：`python manage.py collectstatic`。django会把`STATICFILES_DIRS`中配置的目录下的所有文件和文件夹拷贝到`STATIC_ROOT`配置的目录下。

8、修改与`settings.py`同目录下的`wsgi.py`，添加如下代码：

	import sys
    sys.path.append("/home/myproject")

`"/home/myproject"`替换成项目的根目录，这段代码的作用是把项目的路径添加到python的环境变量下，使得`os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")`这句断码可以正确查找到项目的`settings.py`文件。

# 三、Apache配置：

修改配置文件`httpd.conf`
1）、配置Apache加载mod_wsgi

	LoadModule wsgi_module modules/mod_wsgi.so

2）、配置静态文件映射

    Alias URL-path file-path|directory-path
    如Alias /static "/home/myproject/static"

3）、apache虚拟主机（VirtualHost）配置

```xml
<VirtualHost *:80>
    DocumentRoot /ytxt/surveyproject
    ServerName www.feiyuejihua.com
    WSGIScriptAlias / /ytxt/surveyproject/surveyproject/django.wsgi
    ErrorLog /ytxt/log/feiyuejihua.log
    <Directory />
        Order deny,allow
        Allow from all
    </Directory>
    <Directory /apache>
        Allow from all
    </Directory>
</VirtualHost>
```

* DocumentRoot：为项目的根路径
* ServerName：项目的域名
* ErrorLog：为异常日志路径
* WSGIScriptAlias：映射一个URL到一个文件系统地址并委派目标文件为WSGI Script

# 四、启动服务器：
* 启动：httpd -k start
* 重启：httpd -k restart
* 关闭：httpd -k stop 或 httpd -k shutdown

