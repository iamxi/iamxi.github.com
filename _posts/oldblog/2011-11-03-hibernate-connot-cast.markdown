---
layout: post
title: "Hibernate获取返回值报XXX connot be cast to [Ljava.lang.Object"
date: 2013-07-28 21:41:00
---

今天在测试时候发现在获取Hibernate返回数据时候报java.math.BigDecimal cannot be cast to \[Ljava.lang.Object 异常，但是查看了代码，很简单也很平常，createNativeQuery执行，getResultList获取结果集，每行都用Object\[\]类型。调试时候发现返回结果集为\[2\]，也就是就一行且一个。也就是说，每行的类型不再是 Object\[\]。

按以往， getResultList的返回是一个List(Object\[\])，当如果返回只有一个，比如1或2是，类型就变成了该数据库字段所对应的类型，在我这里就是BigDecimal。

真是不明白，多个就是Object\[\]，一个就变成不是数组了。 Hibernate莫名其妙啊。发下牢骚。。。
