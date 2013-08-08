---
layout: post
title: 《重构》学习笔记-代码的坏味道
date: 2013-07-29 13:09:00
description: 学习《重构 改善既有代码的设计》，认识代码的坏味道。
category: mread
tags: [Refactioring]
keywords: 重构,坏味道
---

代码的坏味道 … 老外没有艺术细胞，作为一名大师级任务，好歹给起个好听点的名称。向金庸老人家学习下，让我们也感染下艺术气息。

我认为吗，坏味道就是对一段问题代码的感觉，从感性慢慢分析到理性。这种感觉需要在实践中培养。观看这里的介绍只能有个大概的理解方向，只有在反复在实际的代码中去体会，才能运用自如。

我也是爱国人士，也很不喜欢英语，不过毕竟都是老外的东西，为了避免一些名词的混淆，我还是用英语加翻译的方式。 

### 一、 重复代码（ Duplicated Code ）

很直观的一个问题，就算新手也很快能看出。如果是同一个类里面有重复代码，使用 Extract Method （提炼函数）把重复的给提取出来；如果重复代码分别在一个类的两个子类中，则先提取代码，然后使用 Pull Up Method （函数上移）把提取的函数放到 superclass 中；如果在好不相干的两个类中，则考虑使用 Extract Class （提炼类）把重复代码独立成一个类，也可以把函数放在其中一个类中，另一个应用这个之，看实际情况了。

### 二、 过长函数（ Long Method ）

看 50 行的 20 个函数比看一千行的一个函数强多了，所以么函数不宜过长，或者说就应该短， java 在调用函数时候的开销很小，一般不会产生性能问题。我也讨厌那些长函数，看到后面逻辑都乱了，本人见过两千行的函数，唉，一半注解一半代码。

99% 的场合只需要使用 Extract Method （提炼函数）。如果函数内有大量的参数和临时变量，可以尝试 Extract Method （提炼函数）把那些参数和临时变量当作参数，运用 Replace Temp With Query （以查询取代临时变量）来消除暂时元素。 Introduce Parameter Object （引入参数对象）和 Preserve Whole Object （保持对象完整）则可以把过长的参数变简洁。如果做完上述发现还是有太多参数和变量，可以使用杀手锏： Replace Method With Method Object （以函数对象取代函数）。条件和循环常常是提取的信号，可以使用 Decompose Conditional （分解条件式）来处理条件式。循环则可以提炼一个独立的函数。

### 三、 过长类（ Large Class ）

这种可以使用 Extract Class （提炼类）提炼出内容有相关性的。如果类里面有数个变量有着一样的前缀或后缀，用 Extract Subclass （提炼子类）会比较简单。如果是 GUI 类，可以把数据和行为移到单独的 domain 对象中去，运用 Duplicate Observed Data （复制被监视数据）同步数据。

### 四、 过长参数列（ Long Parameter List ）

过长的参数列往往难以理解，还可能造成前后不一致，一旦需要更多数据，就不得不修改它。如果“向既有对象发出一条请求”就能取得原本位于参数列上的数据时，就可以使用 Replace Parameter With Method （以函数取代参数）。还可以运用 Preserve Whole Object （保持对象完整）将来自同一对象的一堆数据收集起来，并以该对象替换参数。如果默写参数缺乏合理的对象归属，可以使用 Introduce Parameter Object （引入参数对象）为他们制造出一个参数对象。当然也有例外，如果不希望两个类之间出现依赖关系，可以把数据从对象里面拆解出来，单独作为参数。

### 五、 发散式变化（ Divergent Change ）

发散式变化是指一个类经常因为不同原因在不同的方向上发生变化。这个是违反危险对象的开发原则的“一个类应该有且只有一个改变的理由”。这样的类可以运用 Extract Class （提炼类）将那些同一个原因变化的部分提炼到另一个类中。

### 六、 霰弹式修改（ Shotgun Surgery ）

这个问题恰恰与发散式变化相反，一个变化经常需要去修改多个类。这种问题会导致修改的代码散步四处，出错的几率会变大。可以使用 Move Method （移动函数）和 Move Field （移动值域）把所有需要一同修改的代码放到同一个类。通常可以运用 Inline Class （将类内联化）把一系列相关行为放进同一个类。

### 七、 依恋情结（ Feature Envy ）

好晦涩难懂的名字。这个问题是指一个函数对另一个类的兴趣高过自己所在的类。这种问题很简单，使用 Move Method （移动函数）方法把函数移到它该去的地方。如果只有函数中的部分代码有这个问题，那么先使用 Extract Method （提炼函数），然后在 Move Method （移动函数）。当然也有些设计模式不适合这个原则，比如 Strategy 和 Visitor 。

### 八、 数据泥团（ Data Clumps ）

我的理解是扎堆一起出现的数据（变量、参数等），且在不止一处（不同的类中或是同一个类中）出现，这些应该放进属于他们自己的对象中。在数据值域出现的地方，用 Extract Class （提炼类）将他们提炼到一个独立对象中，然后 Introduce Parameter Object （引入参数对象）或 Preserve Whole Object （保持对象完整）为原有代码减肥。

### 九、 基本型别偏执（ Primitive Obsession ）

看来我语文水平不好，看到这种词我都无法理解。只能慢慢读内容才可以看懂。基本类别偏执：喜欢或钟爱使用基本类型而不愿意使用对象，想着基本类型效率要比使用对象高（至少基本类型小）。不过这样势必带来了一些麻烦，比如太多的变量在函数中。可以运用 Replace Data Value with Object （以对象取代数据值）将原本单独存在的数据替换为对象。如果替换的数据值是 type code （类别码），而它不影响行为，可以使用 Replace Type Code with Class （以类取代型别码）将它换掉。如果有相依于此的 type code 的条件式，可以用 Replace Type Code with Subclass （以子类取代型别码）或 Replace Type Code with State/Strategy （以 State/Strategy 取代型别码）加入处理。如果是一组这样的数据，可以参考数据泥团的解决。

### 十、 Switch 惊悚现身（ Switch Statements ）

Switch 是代码中比较少用到的，但是不是不会用到，一般我也不忌讳用这个。不过书中说面向对象开发中应该少用它，挺费解的。

一看到 switch 就考虑多态来替换。用 Extract Method （提炼函数）将 switch 语句提炼出来，再用 Move Method （移动函数）将它移到需要多态性的那个类里头。此时必须决定是否使用 Replace Type Code with Subclasses （以子类取代型别码）或 Replace Type Code with State/Strategy （以 State/Strategy 取代型别码）。完成继承结构后就可以用 Replace Conditional with Polymorphism （以多态取代条件式）了。

如果只是单一函数中有些选择事例，那就不用多态这把牛刀了。这种情况可以用 Replace Parameter with Explicit Methods （以明确函数取代参数）。如果条件之一是 null ，可以试试 Introduce Null Object （引入 null 对象）。

### 十一、平行继承体系（ Parallel Inheritance Hierarchies ）

这种情况下为某个类增加一个子类的时候，必须同时为另一个类相应的增加一个子类。消除这种重复性的一般策略是：让一个继承体系的实体指涉（参考、引用、 refer to ）另一个继承体系的实体。如果再接再厉，运用 Move Method （提炼函数）和 Move Field （移动值域），就可以将指涉端的继承体系消弭与无形。

### 十二、冗赘类（ Lazy Class ）

### 十三、夸夸其谈未来性（ Speculative Generality ）

### 十四、令人迷惑的暂时值域（ Temporary Field ）

#### 十五、过度耦合的消息链（ Message Chains ）

#### 十六、中间转手人（ Middle Man ）

#### 十七、狎昵关系（ Inappropriate Intimacy ）

#### 十八、异曲同工的类（ Alternative Classes with Different Interfaces ）

#### 十九、不完美的程序库类（ Incomplete Library Class ）

### 二十、纯稚的数据类（ Data Class ）

#### 二十一、被拒绝的遗赠（ Refused Bequest ）

### 二十二、过多的注解（ Comments ） 

实在写不动了。话说这么多坏味道，一个一个记住，不是什么好主意。还是先理解这些，然后在遇到的时候去看下书本。
