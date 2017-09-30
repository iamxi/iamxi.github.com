---
layout: post
title: SpringMVC没有拦截根路径的问题
date: 2014-12-02 15:22:00
description: 在使用tomcat和spring mvc开发web项目时候，月到了根路径无法被拦截器拦截的问题。
tags: [Java]
urlname: springmvc-cant-fiter-root-url
---

在开发tomcat + spring mvc的web工程时候，发现在controller中配置的根路劲不会被拦截到，或者说前段返回的根本就不是controller中所配置的。比如，我在controller中配置根路径的返回为一个欢迎页面：

```java
@RequestMapping(value = "/", method = RequestMethod.GET)
public ModelAndView main() {
    ModelAndView mv = new ModelAndView("main");
    return mv;
}
```

但访问“http://localhost:8080/project_name/”，返回的却是404.

在对该单元测试中，对根路径的访问等得到了正确的结果，这证明spring的配置是正确的。于是乎想起来web.xml中的配置可能存在为题，自然最有可能有问题的是web.xml对默认页面的设置了。及`welcome-file-list`。但是自己并没有配置这个。后来看了tomcat官网对tomcat目录下conf/web.xml的说明才知道，原来此处的web.xml中的配置将被当作默认配置，而部署的工程内的web.xml中的配置会覆盖默认配置，但是如果向我那样没有配置`welcome-file-list`，那么就会使用默认的了。为此当访问根目录的时候，tomcat试图去寻找默认的index.html等资源，而不是把请求交给spring的拦截器。

知道了这些，解决问的就简单了，直接修改工程下的web.xml，加入如下配置：

```xml
<welcome-file-list>
    <welcome-file></welcome-file>
</welcome-file-list>
```

这样就配置后，tomcat就知道不需要对根路径的访问做处理了，而将请求交给了spring的拦截器。