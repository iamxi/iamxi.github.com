---
layout: post
title: Django基础入门
date: 2013-08-29 17:01:00
description: Django基础
category: likeit
tags: [Django]
---

# 安装：
下载后运行python setup.py install或pip安装

# 创建项目：
进入准备创建项目的目录下，运行命令：

	django-admin.py startproject xxx

django-admin.py在windows（其他系统也可能因为安装不同而不在环境变量里）下不能直接调用，可以将其放入环境变量中或是去python的安装目录下的“Lib\site-packages\django\bin”下运行。windows下没有关联文件类型的话，上面的命令前需要加上ptyhon，即

	python django-admin.py startproject xxx

创建的项目结构如下：
{% highlight python  %}
projectname/
|- manage.py
|- projectname/
   |- __init__.py
   |- settings.py
   |- urls.py
   |- wsgi.py
{% endhighlight %}

# 运行项目：

	python manage.py runserver

默认端口号是8000，可以指定端口号运行

	python manage.py runserver 8080

访问http://127.0.0.1:8000/可以显示django的默认页面
这种运行方式并非部署，只是运行Development Server，只能可靠的处理一次单个请求。在此模式下代码或配置的变动都会立即生效。

# 创建APP：

	python manage.py startapp xxx

APP结构如下：

{% highlight python %}
APPname/
|- __init__.py
|- models.py
|- tests.py
|- views.py
{% endhighlight %}

进入项目的settings.py下，把新建的APP加入到INSTALLED\_APPS 中，例如：

{% highlight python %}
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    # 'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'APPname',
)
{% endhighlight %}

# 第一个视图：
在此假定APP的文件夹名是polls

1、打开views.py文件，在polls的目录下，键入示例代码：

{% highlight python %}
from django.http import HttpResponse
def index(request):
     return HttpResponse("Hello world")
{% endhighlight %}

2、在polls下创建urls.py文件，并编辑加入一下代码：

{% highlight python %}
from django.conf.urls import patterns, url
from polls import views
urlpatterns = patterns('',
     url(r'^$', views.index, name='index'))
{% endhighlight %}

3、编辑项目工程的urls.py，即“projectname/projectname/urls.py”，如下：

{% highlight python %}
from django.conf.urls import patterns, include, url
urlpatterns = patterns('',
    url(r'^polls/', include('polls.urls')),
)
{% endhighlight %}

4、浏览器地址栏下键入http://127.0.0.1:8000/polls/

# 模板：
1、在APP目录下新建文件夹templates，并在项目的settings.py文件下配置模版的路径，示例如下：

{% highlight python %}
import os.path
...
TEMPLATE_DIRS = (
    # Put strings here, like "/home/html/django_templates" or "C:/www/django/templates".
    # Always use forward slashes, even on Windows.
    # Don't forget to use absolute paths, not relative paths.
     os.path.join(os.path.dirname(__file__), '../polls/templates').replace('\\','/'),
)
{% endhighlight %}

这边注释已经说的很清楚了，使用绝对路径，如果是windows下使用/代替\。

上面的“os.path.join(os.path.dirname(__file__), '../polls/templates').replace('\\','/'),”是来获取settings.py文件的路径然后拼接出模板文件夹的路径。注意后面添加","。

2、修改views.py文件，如下

{% highlight python %}
from django.shortcuts import render_to_response
def index(request):
     return render_to_response('index.html')
{% endhighlight %}

# 静态文件：

和模板配置类似，在APP目录下新建文件夹static，并在项目的settings.py文件下配置静态文件的路径：

{% highlight python %}
STATICFILES_DIRS = (
     os.path.join(os.path.dirname(__file__), '../polls/static').replace('\\','/'),
)
{% endhighlight %}

# django shell：

一个提供了django的一些api，可以使用项目的上下文的shell，使用命令进入

	python manage.py shell

shell下可以调用一些django的api，模型的操作之类

# 模型：

1、数据库连接设置

在settings.py下，设置DATABASES，示例如下：

{% highlight python %}
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', 
        'NAME': 'mysql', 
        'USER': 'user', 
        'PASSWORD': 'password', 
        'HOST': '127.0.0.1', 
        'PORT': '3306',
    }
}
{% endhighlight %}

ENGINE的不同数据库的设置如下：

* postgres：'django.db.backends.postgresql_psycopg2'
* mysql：'django.db.backends.mysql'
* sqlite：'django.db.backends.sqlite3'
* oracle：'django.db.backends.oracle'

这里python的不同数据库驱动都是要自己安装的，django并不包含这些驱动。

2、模型文件，在app下打开models.py文件，如下：

{% highlight python %}
class Answer(models.Model):
     ans_id=models.AutoField(primary_key=True)
     ans_respondent_id=models.IntegerField()
     ans_question_num=models.IntegerField()
     ans_answer=models.CharField(max_length=100)
{% endhighlight %}

在此，模型是一个类，每个模型都是django.db.models.Model的子类，Model包含了所有必要的数据库操作。一个模型代表着数据库内的一个表，类中属性对应着表的字段。属性的类型相当于数据库的字段类型，如CahrField相当于oracle数据库的varchar2。如果不指定主键，同步的时候django会自动加入一个名为id的自增长主键。

3、模型的同步，django可以使用定义好的模型在数据库内创建对应的表。同步的前提是已经设置好数据库连接，app已经配置到settings.py中。

1）在终端输入命令

	python manage.py validate

该命令可以检查模型的语法和逻辑的正确性。

2）数据库建表语句显示

	python manage.py sqlall appname

会显示创建表的SQL语句,appname是要显示模型所在的app

3）同步

	python manage.py syncdb

该命令执行完成后数据库就会看到模型对应的表了。该语句多次执行也不会重复创建表。

4、基本数据操作。一下操作都是在“python manage.py shell”后执行，
保存

	>>> p1 = Publisher(name='Apress', address='2855 Telegraph Avenue',
	...     city='Berkeley', state_province='CA', country='U.S.A.',
	...     website='http://www.apress.com/')
	>>> p1.save()

查找所有

	>>> publisher_list = Publisher.objects.all()

条件查找（过滤）

	>>> Publisher.objects.filter(name__contains="press")

# 表单：

表单和模型类型，模型对应的是数据库表，而表单对应的是页面的form。app下建forms.py文件，示例代码如下：

{% highlight python %}
class Submitter(forms.Form):
    username=forms.CharField(max_length=15)
    userphone=forms.CharField(max_length=11, min_length=11)
    useremail=forms.EmailField()
    school=forms.CharField(max_length=20)
    age=forms.IntegerField()
    sltype=forms.CharField(max_length=5)
{% endhighlight %}

这边每个表单都是继承django.forms。表单名没有特定要求，而其属性是对应的页面上表单中各个组件的name，属性的类型则是对应组件提交数据的类型。

通过表单提供的方法，可以校验页面所提交的数据，如：

{% highlight python %}
sForm = Submitter(request.POST)
if sForm.is_valid() and qForm.is_valid():
     ur = sForm.cleaned_data
     print(ur['username'])
{% endhighlight %}

上面的cleaned_data属性， 是一个包含干净的提交数据的字典。

表单亦可生成模版代码，即html代码，这样还可以省去编写页面表单的时间。

