---
title: spring cloud feign 拦截器配置
date: 2018-01-25 20:53:21
tags: [java, spring cloud]
urlname: spring-cloud-feign-add-Interceptor
---

# 问题
公司大规模推全链路，要求所有应用都接入。手头一个新的项目，采用spring cloud框架。通过 `feign` 调用rest接口。但是支持提供的支持 `HttpClient4` 的方式无法对 `feign` 起作用。

# 思路
想着 `feign` 也是采用 `HttpClient` 的。按理对 `HttpClient4` 起作用的也应该同样有效。然后去看了下架构组全链路配置代码，原来他们是在初始化 `HttpCLient` 实例时候，加入了拦截器。通过拦截器，在 `header` 中加入链路信息。但是这个方式似乎对 feign 没啥作用。那我就自己实现添加拦截器呗。

查了下 feign 添加拦截器，网络上提供的方式都是通过自定义 `FeignConfiguration` 的方式来实现。方法如下：

新建 `FeignConfiguration`类

```java
public class FeignConfiguration {

    @Bean
    public FeignBasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new FeignBasicAuthRequestInterceptor();
    }
}
```

然后在配置客户端调用时候加入配置中

```java
@FeignClient(value = "xxx-service", path = "/xxx", configuration = FeignConfiguration.class)
```

只是这种方法无法做到全局。找了好久，看了文档，都没有办法配置全局。

无奈，去翻源码吧。找到了 `FeignAutoConfiguration` 类。看到了初始化配置。

```java
protected static class HttpClientFeignConfiguration {
    @Autowired(
        required = false
    )
    private HttpClient httpClient;

    protected HttpClientFeignConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean({Client.class})
    public Client feignClient() {
        return this.httpClient != null ? new ApacheHttpClient(this.httpClient) : new ApacheHttpClient();
    }
}
```

上面代码可以看到，会先去尝试注入已有的httpClient，如果没有，这使用默认的。这么问题就简单了。

# 解决

初始化一个自定义的httpClient提供feign使用。

```java
@Bean
public HttpClient httpClient(Tracer tracer, SpanHttpclientManager spanHttpclientManager) {
    CloseableHttpClient httpClient = HttpClientBuilder
            .create()
            .addInterceptorLast(new TracingHttpclientRequestInterceptor(tracer, spanHttpclientManager))
            .addInterceptorLast(new TracingHttpclientResponseInterceptor(tracer, spanHttpclientManager))
            .build();
    return httpClient;
}
```

问题解决。不过不知道是不是最佳方法。