---
layout: post
title: Java可变参数方法及它的重载
date: 2013-07-26 12:00:00
description: Java可变参数方法及它的重载
---

JAVA支持方法的可变参数。类似如下
{% highlight java %}
public void method1(String... arg1);
{% endhighlight %}
这样，调用method1时候，参数的数量是可变的，如method1("1","2","3")。注意可变参数只能是最后一个参数。

JAVA内部是把这样的参数看成一个数组来处理。在获取多个参数的时候可以用数组遍历来完成。
{% highlight java %}
for (String temp : arg1) {  
      System.out.println("one of the arguments is " + temp);  
}
{% endhighlight %}
可能有人会说，这样的循环遍历像List那样的集合也可以，凭什么说是数组。那好，再看下面代码：
{% highlight java %}
public void method1(String... arg1);  
public void method1(String[] arg1);
{% endhighlight %}
这样的重载，在编译时候会报错，JAVA不允许这样的重载，其实理由很简单，String... 和String[]对编译器来说是一样的。参数一样自然无法重载。

下面是可变参数方法的重载：
{% highlight java %}
public void method1(String... arg1) {  
}  
  
public void method1(String a, String... arg1) {  
}
{% endhighlight %}
我在eclipse中测试，IDE并没有提示错误。但是在实例后调用就不行了，IDE报“The method method1(String[]) is ambiguous for the type ...”，大致是说这个方法有混淆，无法正确的调用。话说类编译的时候没问题，到实例后调用就会提示错误。
{% highlight java %}
public void method1(String... arg1) {  
    System.out.println("1");  
}  
  
public void method1(String a, String b) {  
    System.out.println("2");  
}
{% endhighlight %}
上述是完全可以的，但是如果方法被调用是输入的参数是2个的时候，到底上面哪个会被调用呢。答案是参数固定那个。能匹配定长的方法，那么优先匹配该方法。含有不定参数的那个重载方法是最后被选中的。


之前对可变参数方法及这种方法的重载也就个模糊的影像，在开发中偶尔用下，昨天遇到了个奇怪的问题。

定义了两个方法：
{% highlight java %}
protected List executeQueryMultiplyResult(String sql,
    Object... params);  
protected List executeQueryMultiplyResult(String sql,
    Class<T> arg, Object... params);
{% endhighlight %}
一开始很正常，比如executeQueryMultiplyResult("sql", a.class, new Object[])，这种调用能正确的调用上述的第二种。昨天，因为设计的需要，把这两个方法pull up到了父类中去。结果就有了问题，有些能正确调用，有些却不能。如果都不能，也就当时编译器无法正确区分这两个。网上差了好多可变参数方法的重载问题，我都没有犯那些错误。调试了半天，后来发现在IDE代码只能不全的时候提示第二种方法的第二个参数是固定的。自己研究了下代码，原来Class<T> arg中的T是泛型，子类在继承父类时指定了这个泛型。所以这边，实际arg参数实际已经固定了。如果子类的T是类a，那么如果传a.class，就会选择第二个方法，如果传入了其他，如b.class，就变了第一种方法，Object也是包含了Class的。唉，滥用泛型导致的错误啊。
