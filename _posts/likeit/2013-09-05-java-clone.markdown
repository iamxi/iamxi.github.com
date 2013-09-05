---
layout: post
title: Clone使用方法详解
date: 2013-09-05 13:29:00
description: 在实际编程过程中，我们常常要遇到这种情况：有一个对象A，在某一时刻A中已经包含了一些有效值，此时可能会需要一个和A完全相同新对象B，并且此后对B 任何改动都不会影响到A中的值，也就是说，A与B是两个独立的对象，但B的初始值是由A对象确定的。在Java语言中，用简单的赋值语句是不能满足这种需 求的。要满足这种需求虽然有很多途径，但实现clone（）方法是其中最简单，也是最高效的手段。
category: likeit
tats: [Java]
---

Java语言的一个优点就是取消了指针的概念，但也导致了许多程序员在编程中常常忽略了对象与引用的区别。和C语言一样，当把Java的基本数据类型（如int，char，double等）作为 入口参数传给函数体的时候，传入的参数在函数体内部变成了局部变量，这个局部变量是输入参数的一个拷贝，所有的函数体内部的操作都是针对这个拷贝的操作， 函数执行结束后，这个局部变量也就完成了它的使命，它影响不到作为输入参数的变量。这种方式的参数传递被称为"值传递"。而在Java中用对象的作为入口 参数的传递则缺省为"引用传递"，也就是说仅仅传递了对象的一个"引用"，这个"引用"的概念同C语言中的指针引用是一样的。当函数体内部对输入变量改变 时，实质上就是在对这个对象的直接操作。

# java中的clone：

在实际编程过程中，我们常常要遇到这种情况：有一个对象A，在某一时刻A中已经包含了一些有效值，此时可能会需要一个和A完全相同新对象B，并且此后对B 任何改动都不会影响到A中的值，也就是说，A与B是两个独立的对象，但B的初始值是由A对象确定的。在Java语言中，用简单的赋值语句是不能满足这种需 求的。要满足这种需求虽然有很多途径，但实现clone（）方法是其中最简单，也是最高效的手段。

Java的所有类都默认继承java.lang.Object类，在java.lang.Object类中有一个方法clone()。JDK API的说明文档解释这个方法将返回Object对象的一个拷贝。要说明的有两点：

** 一是拷贝对象返回的是一个新对象，而不是一个引用。**

** 二是拷贝对象与用 new操作符返回的新对象的区别就是这个拷贝已经包含了一些原来对象的信息，而不是对象的初始信息。**

有三个值得注意的地方，

一是希望能实现clone功能的CloneClass类实现了Cloneable接口，这个接口属于java.lang包， java.lang包已经被缺省的导入类中，所以不需要写成java.lang.Cloneable。

另一个值得请注意的是重载了clone()方法。最 后在clone()方法中调用了super.clone()，这也意味着无论clone类的继承结构是什么样的，super.clone()直接或间接调 用了java.lang.Object类的clone()方法。下面再详细的解释一下这几点。

应该说第三点是最重要的，仔细观察一下Object类的clone()一个native方法，native方法的效率一般来说都是远高于java中的非 native方法。这也解释了为什么要用Object中clone()方法而不是先new一个类，然后把原始对象中的信息赋到新对象中，虽然这也实现了 clone功能。对于第二点，也要观察Object类中的clone()还是一个protected属性的方法。这也意味着如果要应用clone()方 法，必须继承Object类，在Java中所有的类是缺省继承Object类的，也就不用关心这点了。然后重载clone()方法。还有一点要考虑的是为 了让其它类能调用这个clone类的clone()方法，重载之后要把clone()方法的属性设置为public。那么clone类为什么还要实现Cloneable接口呢？

稍微注意一下，Cloneable接口是不包含任何方法的！其实这个接口仅仅是一个标志，而且 这个标志也仅仅是针对Object类中clone()方法的，如果clone类没有实现Cloneable接口，并调用了Object的clone()方法（也就是调用了super.Clone()方法），那么Object的clone()方法就会抛出 CloneNotSupportedException异常。

# 什么是影子clone？

调用Object类中clone()方法产生的效果是：先在内存中开辟一块和原始对象一样的空间，然后原样拷贝原始对象中的内容。对基本数据类型，这样的操作是没有问题的，但对非基本类型变量，我们知道它们保存的仅仅是对象的引用，这也导致clone后的非基本类型变量和原始对象中相应的变量指向的是同一个对象。


# 怎么进行深度clone？

需要两个改变，一是让对象内的非基本类型变量也实现Clone功能（实现Cloneable接 口，重载clone()方法）。二是在对象的Clone方法中加一句o.unCA = (UnCloneA)unCA.clone();

要知道不是所有的类都能实现深度clone的。例如，如果把上面的CloneB类中的UnCloneA类型变量改成StringBuffer类型，看一下 JDK API中关于StringBuffer的说明，StringBuffer没有重载clone()方法，更为严重的是StringBuffer还是一个 final类，这也是说我们也不能用继承的办法间接实现StringBuffer的clone。

还要知道的是除了基本数据类型能自动实现深度clone以外，String对象，Integer，Double等是一个例外，它clone后的表现好象也实现了深度clone，虽然这只是一个假象，但却大大方便了我们的编程。

Clone中String和StringBuffer的区别,这个区别主要原因来自String和StringBuffer的区别。

