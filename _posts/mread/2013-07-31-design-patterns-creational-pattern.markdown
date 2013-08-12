---
layout: post
title: 设计模式学习-创建型模式
date: 2013-07-31 09:59:00
description: 用于创建对象的模式。包括单件模式（Singleton）、抽象工厂模式（Abstract Factory）、生成器模式（Builder）、工厂方法模式（Factory Method）、原型模式（Prototype）。
category: mread
tags: ["Design Patterns"]
---

### 创建型模式：

用于创建对象的模式。包括[单件模式（Singleton）](/blog/2013/07/31/design-patterns-singleton)、[抽象工厂模式（Abstract Factory）](/blog/2013/07/30/Design-Patterns-abstract-factory)、[生成器模式（Builder）](/blog/2013/07/30/design-patterns-builder)、[工厂方法模式（Factory Method）](/blog/2013/07/31/design-patterns-factory-method)、[原型模式（Prototype）](/blog/2013/07/31/design-patterns-prototype)。

用一个系统创建的那些对象的类对系统进行参数化有两种常用方法：

一、生成创建对象的类的子类；这对应于使用 Factory Method模式。这种方法的主要缺点是，仅为了改变产品类，就可能需要创建一个新的子类。这样的改变可能是级联的。

二、对系统进行参数化的方法更多的依赖于对象复合：定义一个对象负责明确产品对象的类，并将它作为该系统的参数。这是 Abstract Factory、Builder和Prototype模式的关键特征。所有这三个模式都涉及到创建一个新的负责创建产品对象的“工厂对象”。Abstract Factory由这个工厂对象产生多个类的对象。 Builder由这个工厂对象使用一个相对复杂的协议，逐步创建一个复杂产品。 Prototype由该工厂对象通过拷贝原型对象来创建产品对象。

### 如何选择：

使用Abstract Factory、Prototype或Builder的设计甚至比使用Factory Method的那些设计更灵活，但它们也更加复杂。通常，设计以使用 Factory Method开始，并且当设计者发现需要更大的灵活性时，设计便会向其他创建型模式演化。

当然没必要就不要用。

