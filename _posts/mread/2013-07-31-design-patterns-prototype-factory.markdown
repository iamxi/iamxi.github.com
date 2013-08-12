---
layout: post
title: 原型工厂模式
date: 2013-07-31 13:58:00
description: 顾名思义就是原型模式和工厂模式的结合。在GOF的书里面，抽象工厂那节中对这个有描述，不过内容很少，所以看的时候也没注意到。
category: mread
tags: ["Design Patterns"]
---

顾名思义就是[原型模式](/blog/2013/07/31/design-patterns-prototype)和[工厂模式](/blog/2013/07/31/design-patterns-factory-method)的结合。在GOF的书里面，抽象工厂那节中对这个有描述，不过内容很少，所以看的时候也没注意到。

说个实际的问题：系统需要向用户发送短信，但是根据场景的不同，发送的短信格式也不同，比如重置密码和发送临时验证码的短信格式就有区别，而之前的开发人员使用了策略模式，每个内容都有一个类，现在也不方便去改动这些。但是随需求的增加，发送的场景在不断增加，策略+工厂方法模式的情况下需要去增加更多的if-else，导致扩展困难，也使得调用时候不直观。

本人自认为自己小脑瓜没有能力想出好办法，只好去翻下书籍，无意中看到了抽象工厂内介绍原型工厂的内容，想着可以借鉴。如果依靠一定规范的传入参数，通过if-else来判断需要使用的算法闲代码忒长，那么不为什么不直接传入算法呢。当然传入算法的类的话太费资源，但是传个Class或Class Name不适问题。

参数就是用枚举，这比String参数好，原因是String可以传入任何字符串，而枚举只能是枚举内的指定值，枚举代码如下：

{% highlight java %}
public enum MessageBuilderEnum {  
  
    RESET_PASSWORD_SMS(ResetPwSmsMsgSendServiceImpl.class),  
    RESET_PASSWORD_EMAIL(ResetPwMailMsgSendServiceImpl.class),  
    SMS_FEE(SmsFeeMsgSendServiceImpl.class),  
    TEMP_CODE_SMS(TempCodeSmsMsgSendServiceImpl.class),  
    UNIVERSAL_SMS(BoundMsgSendServiceImpl.class);  
      
    private Class<?> builderClass;  
      
    MessageBuilderEnum(Class<?> builderClass) {  
        this.builderClass = builderClass;  
    }  
      
    public Class<?> getBuidlerClass() {  
        return builderClass;  
    }  
}
{% endhighlight %}
<br />
再看下功能实现的代码
{% highlight java %}
private static ConcurrentHashMap<String,
    AbstractMsgSendServiceImpl> messageBuilder =
    new ConcurrentHashMap<String, AbstractMsgSendServiceImpl>();  
  
public MessageSendServiceImpl() {  
    for (MessageBuilderEnum builderClass 
        : MessageBuilderEnum.values()) {  
        try {  
            messageBuilder.put(builderClass.getBuidlerClass()
                .getSimpleName(),  
                    (AbstractMsgSendServiceImpl) builderClass
                        .getBuidlerClass().newInstance());  
        } catch (InstantiationException e) {  
            logger.error(e.getMessage(), e);  
        } catch (IllegalAccessException e) {  
            logger.error(e.getMessage(), e);  
        }  
    }  
}  
  
@Override  
public String send(ServMessage message,
    MessageBuilderEnum builderEnum) {  
    AbstractMsgSendServiceImpl builder = messageBuilder  
            .get(builderEnum.getBuidlerClass()
            .getSimpleName()).clone();  
    if (builder != null) {  
        return builder.send(message);  
    } else {  
        return null;  
    }  
}
{% endhighlight %}
<br />
工厂方法是在构造函数下，在初始化的时候就为枚举下所有的类初始化，然后保存在Map下，用于当作原型。在需要使用的时候再对原型clone一下就是了。

其实可以在需要使用时候初始化。这里使用原型是听说原型创建新对象比较快，当然本人没仔细测试过，有时间再研究。

原型工厂的工厂可以是抽象工厂模式，也可以是工厂方法模式。
