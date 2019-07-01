---
title: zuul网关丢失鉴权信息问题
date: 2018-03-22 15:43:46
tags: [feign, zuul, spring cloud]
urlname: zuul-gateway-lost-auth-info
---

公司有个项目使用spring cloud全家桶。由zuul做入口，路由到各个业务系统。其中路由完成登陆鉴权。后来有一个业务系统需要用到这些登陆信息，所以希望路由这端将登陆的信息通过消息头的方式带到业务系统。

想想实现很简单，定义一个feign的拦截器，把登陆信息填到header中。不过中间遇到个小问题。统一的路由中将信息保持到了一个`ThreadLocal`中，拦截器会从这个`ThreadLocal`中取出数据。问题就出在这里，取到的数据是空的。其他地方也在使用这个`ThreadLocal`中的数据，并未发现问题，未读这个feign拦截器却不行。最奇怪的本地测试可以，跑测试环境就不行。

再三排查，也查了不少资料，最后看到有博客上提到差不多的问题，原因是`Hystrix`的策略导致的。`Hystrix`使用不同的线程完成工作，也就是说`Hystrix`的线程与Request不再同一个线程中。这样，基于`ThreadLocal`的一些功能都无法正常使用。后来再stackoverflow上找到了解决办法，[连接](https://stackoverflow.com/questions/34719809/unreachable-security-context-using-feign-requestinterceptor)。框架的开发者也知道这问题，所以提供了一套完整的解决方案。`HystrixRequestContext`和`HystrixRequestVariableDefault`就是用来解决这个跨线程时共享数据问题的。简单的说，放入`HystrixRequestVariableDefault`中的数据，不管是在Requst线程中，还是在Hystrix的线程中，都可以访问到。不过`HystrixRequestVariableDefault`必须在每一个请求开始的时候进行初始化，同时在请求结束收回收里面的数据。

方案有了，上代码，先实现一个mvc的拦截器，将`ThreadLocal`中的数据转到`HystrixRequestVariableDefault`中：

```java
public class HystrixContext {

    public static final String LOGIN_INFO_HEADER_KEY = "his-login-info";
    public static final String TOKEN_HEADER_KEY = "token";
    public static final HystrixRequestVariableDefault<LoginInfo> LOGIN_INFO = new HystrixRequestVariableDefault<>();

}
```

```java
try {
    if (!HystrixRequestContext.isCurrentThreadInitialized()) {
        HystrixRequestContext.initializeContext();
    }

    LoginInfo loginInfo = convertLoginInf(request.getHeader(HystrixContext.LOGIN_INFO_HEADER_KEY));
    LoginInfoUtil.setInfo(loginInfo);

    String token = request.getHeader(HystrixContext.TOKEN_HEADER_KEY);
    LoginInfoUtil.setToken(token);

    HystrixContext.LOGIN_INFO.set(LoginInfoUtil.getLoginInfo());
} catch (Exception e) {
    LOG.error("从头信息中获取数据失败", e);
}
```

然后给feign加一个拦截器：

```java
try {
    if (!HystrixRequestContext.isCurrentThreadInitialized()) {
        HystrixRequestContext.initializeContext();
    }

    LoginInfo loginInfo = HystrixContext.LOGIN_INFO.get();
    if (loginInfo != null) {
        template.header(HystrixContext.TOKEN_HEADER_KEY, loginInfo.getToken());
        template.header(HystrixContext.LOGIN_INFO_HEADER_KEY, Base64.getUrlEncoder().encodeToString(JSON.toJSONString(loginInfo).getBytes()));
    }
} catch (Exception e) {
    LOG.error("从头信息中获取数据失败", e);
}
```

## HystrixRequestVariableDefault的原理
查看了下`HystrixRequestVariableDefault`的源码（下面源码都使用hystrix-core 1.5.12版本），`HystrixRequestVariableDefault`中并没有维护任何数据，而是将维护数据的工作交给了`HystrixRequestContext`，自己则只是封装了`get`、`set`方法。

看下`HystrixRequestContext`的实现，`HystrixRequestContext`维护两个属性及其方法:
```java
private static ThreadLocal<HystrixRequestContext> requestVariables = new ThreadLocal<HystrixRequestContext>();

ConcurrentHashMap<HystrixRequestVariableDefault<?>, HystrixRequestVariableDefault.LazyInitializer<?>> state = new ConcurrentHashMap<HystrixRequestVariableDefault<?>, HystrixRequestVariableDefault.LazyInitializer<?>>();

public static HystrixRequestContext initializeContext() {
    HystrixRequestContext state = new HystrixRequestContext();
    requestVariables.set(state);
    return state;
}

public static HystrixRequestContext getContextForCurrentThread() {
    HystrixRequestContext context = requestVariables.get();
    if (context != null && context.state != null) {
        return context;
    } else {
        return null;
    }
}

public static void setContextOnCurrentThread(HystrixRequestContext state) {
    requestVariables.set(state);
}
```

这里的重点是初始化，初始化的过程是实例化一个`HystrixRequestContext`，存到当前线程数据中。这样每个线程关联一个HystrixRequestContext，每个HystrixRequestContext有个Map结构（`ConcurrentHashMap<> state`）存储数据。

那问题来了，谁来负责在线程间拷贝这份数据呢？答案在`HystrixContextRunnable`和`HystrixContextCallable`中。

`HystrixContextRunnable`在实现了Runnalbe接口：

```java
public void run() {
    HystrixRequestContext existingState = HystrixRequestContext.getContextForCurrentThread();
    try {
        // set the state of this thread to that of its parent
        HystrixRequestContext.setContextOnCurrentThread(parentThreadState);
        // execute actual Callable with the state of the parent
        try {
            actual.call();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    } finally {
        // restore this thread back to its original state
        HystrixRequestContext.setContextOnCurrentThread(existingState);
    }
}
```

`HystrixContextRunnable`的使用可以在`HystrixCommand`找到，关于`HystrixCommand`又是另一个话题 —— Hystrix的命令模式，不展开（主要是我还没研究清楚）。

## HystrixRequestVariableDefault使用需要注意的事
1. 使用前需要用`HystrixRequestContext.initializeContext()`进行初始化
2. 结束时需使用`HystrixRequestContext.shutdown()`进行清理
3. 父线程调用shutdown时，子线程的HystrixRequestVariables也会被清理