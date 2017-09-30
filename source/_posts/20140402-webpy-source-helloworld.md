---
layout: post
title: webpy源码阅读之HelloWorld
date: 2014-04-03 11:21:00
description: 学习webpy框架
tags: [web.py, Python]
urlname: webpy-source-helloworld
---

使用过Django和web.py，只是一直没有好好深入学习过，Django使用时间比较长，不过Django庞大了点，不适合我这种没耐心的人去学习，所以选择了web.py。

## Hello World!

先看下web.py官方cookbook的“Hello World!”例子

```python
import web

urls = ("/.*"), "hello")
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, world!'

if __name__ == "__main__":
    app.run()
```

在例子中可以看到app.run()是启动了服务，而app的是application的一个实例，初始化工作是web.application(urls, globals())，就拿这个做入口，来看下application类的源码吧。

## 初始化

初始化的流程：

* 初始化mapping
* 把\_load和\_unload变成钩子加入到processors
* 把Reloader和reload\_mapping变成钩子加入到processors
* 载入main所在模块

init的参数有三个，第一个mapping，上面例子中的urls；第二个fvars，例子中的globals()，globals()是python自带的一个函数，其返回值是全局名字空间，包括了函数、类、导入模块等等，是一个字典类型；第三个autoreload，是否允许自动重新载入。

### mapping初始化

```python
self.init_mapping(mapping)
def init_mapping(self, mapping):
	self.mapping = list(utils.group(mapping, 2))
```

最后mapping的形式会是[['1','2'],['1','2']]这样，这边就可以理解urls列表的样式了。

### \_load和\_unload变成钩子加入到processors

关于钩子和处理器详见[官网](http://webpy.org/cookbook/application_processors.zh-cn)。

```python
self.add_processor(loadhook(self._load))
self.add_processor(unloadhook(self._unload))
```

loadhook函数的作用就是在执行其参数传入的函数前执行一些操作，unloadhook则是在其后执行一些操作。这两个都类似于python的decorator。而\_load和\_unload则是将application实例本身加入web.ctx.app\_stack或从其中移除。

***如果autoreload不为True则初始化就到此为止了。在初始化的时候，如果autoreload为None，则回去参数读取web.config内的debug的值，默认为False，所以如果autoreload参数没有传入，一般都是不会自动加载，后面操作不会再去进行了。***

### Reloader和reload\_mapping变成钩子加入到processors

和第二步相似，reload\_mapping就是用来从新载入主的app，并初始化mapping。而Reloader则是在检查磁盘上任何模块是否有变化，如果有则重新加载。

### 重新import

通过main获取主app所在的模块名称和所在文件名，然后通过文件名重新import。这步还是简单的，不过代码值得一看，可以学好如sys.modules、getattr()及\_\_import\_\_()的用法。

## 启动服务

```python
def run(self, *middleware):
	return wsgi.runwsgi(self.wsgifunc(*middleware))
```

run函数调用wsgi模块的runwsgi函数，在hello world中就是启动一个http simple服务。具体内容等看到wsgi模块和httpserver模块再说了。
