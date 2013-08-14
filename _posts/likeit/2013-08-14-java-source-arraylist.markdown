---
layout: post
title: Java源码解读——ArrayList
date: 2013-08-14 15:11:00
description: ArrayList是一个实现可变长数组，继承AbstractList类，实现所有的List接口，还实现了RandomAccess、Cloneable、Serializable接口。ArrayList不进行同步，除此之外基本和Vector等同。
category: likeit
tags: [Java]
---

为了提高自己的Java开发能力，我也向高手、牛人学习，去解读源码。自己底子差了点，不过看个源码还是没问题的。第一站ArrayList。

**源码为Java 1.7的源码**

ArrayList是一个实现可变长数组，继承AbstractList类，实现所有的List接口，还实现了RandomAccess、Cloneable、Serializable接口。ArrayList不进行同步，除此之外基本和Vector等同。

## 1、成员变量
	private transient Object[] elementData;

elementData用于保存数据的数组。

	private int size;

size为ArrayList内的数据数量，但并不是elementData的长度。

## 2、构造方法 
{% highlight java %}
public ArrayList(int initialCapacity) {  
    super();  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal Capacity: "+  
                                           initialCapacity);  
    this.elementData = new Object[initialCapacity];  
}  
  
public ArrayList() {  
    this(10);  
}  
  
public ArrayList(Collection<? extends E> c) {  
    elementData = c.toArray();  
    size = elementData.length;  
    if (elementData.getClass() != Object[].class)  
        elementData = Arrays.copyOf(elementData, size, Object[].class);  
}
{% endhighlight %}

第一个是根据指定长度建立List，第二个是根据默认长度建立List，第三个根据传入Collection子类实例创建List。通过第三个实例看到Collection子类都会实现toArray()方法，可以看出至少很多Collection子类依赖于数组来实现（还没看过其他的集合实现，这只是我的妄言而已）。

## 3、add方法
{% highlight java %}
public boolean add(E e) {  
    ensureCapacityInternal(size + 1);   
    elementData[size++] = e;  
    return true;  
}
{% endhighlight %}

该方法在数组最后添加一个元素。ensureCapacityInternal方法提供了ArrayList的自增长实现，以确保elementData有足够的长度来容纳新进入的元素（后面介绍自增长的实现）。

{% highlight java %}
public void add(int index, E element) {  
    rangeCheckForAdd(index);  
  
    ensureCapacityInternal(size + 1);   
    System.arraycopy(elementData, index, elementData, index + 1,  
                     size - index);  
    elementData[index] = element;  
    size++;  
}
{% endhighlight %}

该方法是在第i个位置插入一个元素。rangeCheckForAdd方法用于检查index是否越界，代码如下：

{% highlight java %}
private void rangeCheckForAdd(int index) {  
    if (index > size || index < 0)  
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
}
{% endhighlight %}

System.arraycopy方法使用native修饰符（这个修饰符第一次看到，谷歌了下，大致是说这个修饰的方法是通过底层实现的），效率自然高。arraycopy如果对同一个数组进行操作时，会首先把从源部分拷贝到一个临时数组，在把临时数组的元素拷贝到目标位置。值得注意的是，如果数组内不是基本类型，System.arraycopy的拷贝过来的也只是个引用而已。

## 4、remove方法
{% highlight java %}
public E remove(int index) {  
    rangeCheck(index);  
  
    modCount++;  
    E oldValue = elementData(index);  
  
    int numMoved = size - index - 1;  
    if (numMoved > 0)  
        System.arraycopy(elementData, index+1, elementData, index,  
                         numMoved);  
    elementData[--size] = null;  
  
    return oldValue;  
}
{% endhighlight %}

 删除指定位置的元素，并返回删除的元素。以前还真没注意过删除时候还返回删除的元素。rangeCheck方法用于检查越界，代码如下：

{% highlight java %}
private void rangeCheck(int index) {  
    if (index >= size)  
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));  
}
{% endhighlight %}

modCount用于记录ArrayList的结构性变化的次数，add()、remove()、addall()、removerange()及clear()方法都会让modCount增长。add()及addall()方法的对modCount的操作在ensureCapacityInternal中。

numMoved用于检查删除的元素是否是最后ArrayList的最后一个元素（不一定是数组elementData的最后一个）。如果不是最后一个，则用arraycopy移动数组，最后将数组size-1处赋值为空。

{% highlight java %}
public boolean remove(Object o) {  
    if (o == null) {  
        for (int index = 0; index < size; index++)  
            if (elementData[index] == null) {  
                fastRemove(index);  
                return true;  
            }  
    } else {  
        for (int index = 0; index < size; index++)  
            if (o.equals(elementData[index])) {  
                fastRemove(index);  
                return true;  
            }  
    }  
    return false;  
}
{% endhighlight %}

 删除与Object o相同的第一个元素。这里fastRemove方法，从代码来看和remove(int index)类似。

{% highlight java %}
private void fastRemove(int index) {  
    modCount++;  
    int numMoved = size - index - 1;  
    if (numMoved > 0)  
        System.arraycopy(elementData, index+1, elementData, index,  
                         numMoved);  
    elementData[--size] = null;  
}
{% endhighlight %}

## 5、addAll方法
{% highlight java %}
//将Collection c内的数据插入ArrayList中  
public boolean addAll(Collection<? extends E> c) {  
     Object[] a = c.toArray();  
     int numNew = a.length;  
     ensureCapacityInternal(size + numNew);  // Increments modCount  
     System.arraycopy(a, 0, elementData, size, numNew);  
     size += numNew;  
     return numNew != 0;  
 }  
  
 //将Collection c中的数据插入到ArrayList的指定位置  
 public boolean addAll(int index, Collection<? extends E> c) {  
     rangeCheckForAdd(index);  
  
     Object[] a = c.toArray();  
     int numNew = a.length;  
     ensureCapacityInternal(size + numNew);  // Increments modCount  
  
     int numMoved = size - index;  
     if (numMoved > 0)  
         System.arraycopy(elementData, index, elementData, index + numNew,  
                          numMoved);  
  
     System.arraycopy(a, 0, elementData, index, numNew);  
     size += numNew;  
     return numNew != 0;  
 }
{% endhighlight %}

这两个方法都会调用ensureCapacityInternal方法使数组能够容纳下新的数据。后一个方法会判断是否是在数据最后插入，如果是则和第一个方法相同，如果不是，则先使用arraycopy移动数组。

## 6、removeAll和retainAll

删除或保留ArrayList中包含Collection c中的的元素，这两个方法都依赖batchRemove(Collection<?> c, boolean complement)实现。

## 7、get和set

前者是获取元素，后者是替换某个位置上的元素，然后返回被替换的元素。这两个都用rangeCheck做越界检查。set方法不进行自增长，也就说替换的元素必须是size内的。

## ArrayList的自动变长机制
都知道ArrayList不像数组那样是定长的，然而ArrayList也使用了数组来保存数据，所以么，自然很关心是怎么实现变长的。

ArrayList通过ensureCapacityInternal(int minCapacity)方法实现自身容量的增加，在add()和addAll()方法里面都调用了改方法。

{% highlight java %}
private void ensureCapacityInternal(int minCapacity) {  
    modCount++;  
    if (minCapacity - elementData.length > 0)  
        grow(minCapacity);  
}
{% endhighlight %}

方法名的英文大致意思是“确保内部容量”，这里要说明，size表示的是现有的元素个数，并非ArrayList的容量，容量应该是数组elementData的长度。参数minCapacity是需要检查的最小容量，即方法的功能就是确保elementData的长度不小于minCapacity，如果不够，则调用grow增加容量。容量的增长也算结构性变动，所以modCount需要加1。

grow代码：
{% highlight java %}
private void grow(int minCapacity) {  
    int oldCapacity = elementData.length;  
    int newCapacity = oldCapacity + (oldCapacity >> 1);  
    if (newCapacity - minCapacity < 0)  
        newCapacity = minCapacity;  
    if (newCapacity - MAX_ARRAY_SIZE > 0)  
        newCapacity = hugeCapacity(minCapacity);  
    elementData = Arrays.copyOf(elementData, newCapacity);  
}  
  
private static int hugeCapacity(int minCapacity) {  
    if (minCapacity < 0)   
        throw new OutOfMemoryError();  
    return (minCapacity > MAX_ARRAY_SIZE) ?  
        Integer.MAX_VALUE :  
        MAX_ARRAY_SIZE;  
}
{% endhighlight %}

先对容量扩大1.5倍，这里oldCapacity >> 1是二进制操作右移，相当于除以2，如果不知道这个面壁去吧（刚看到时候也没反应过来这个是什么操作，毕竟平时java中很少用到左移、右移这种运算，没事谢个测试用例测试下计算效果就知道是干什么的了）。接再来把新的临时容量（还没正式改变容量，应该叫预期容量）和实际需要的最小容量比较，如果还不满足，则把临时容量改成需要的最小容量值。在判断容量是否超过MAX\_ARRAY\_SIZE的值，MAX\_ARRAY\_SIZE值为Integer.MAX\_VALUE - 8，比int的最大值小8，不知道为什设计，可能方便判断吧。如果已经超过，调用hugeCapacity方法检查容量的int值是不是已经溢出。一般很少用到int最大值的情况，那么多数据也不会用ArrayList来做容器了，估计这辈子没机会见到hugeCapacity运行一次了。最后确定了新的容量，就使用Arrays.copyOf方法来生成新的数组，copyOf也已经完成了将就的数据拷贝到新数组的工作。

当然有增就有减，trimToSize()就是用来将elementData的长度变成size。

{% highlight java %}
public void trimToSize() {  
    modCount++;  
    int oldCapacity = elementData.length;  
    if (size < oldCapacity) {  
        elementData = Arrays.copyOf(elementData, size);  
    }  
}
{% endhighlight %}
这样可以解决平凡新增、删除元素后elementData过大的问题。

从上面的操作来看，ArrayList的容量增长的开始还是有的，就算那些数字的计算忽略不计，Arrays.copyOf的操作的消耗也不会小。所以在初始化ArrayList的时候尽量预算下大致的容量需求，降低平凡调整容量的开销。

## 序列化
ArrayList实现了Serializable接口，所以ArrayList可以进行序列化。但是elementData的定义

	private transient Object[] elementData;

transient修饰符让elementData无法自动序列化，这样的原因是，数组内存储的的元素其实只是一个引用，单单序列化一个引用没有任何意义，反序列化后这些引用都无法在指向原来的对象。ArrayList使用writeObject()实现手工序列化数组内的元素。

{% highlight java %}
private void writeObject(java.io.ObjectOutputStream s)  
    throws java.io.IOException{  
    int expectedModCount = modCount;  
    s.defaultWriteObject();  
  
    s.writeInt(elementData.length);  
  
    for (int i=0; i<size; i++)  
        s.writeObject(elementData[i]);  
  
    if (modCount != expectedModCount) {  
        throw new ConcurrentModificationException();  
    }  
}
{% endhighlight %}

## 迭代器实现
一般ArrayList需要遍历时，可以调用他的iterator()方法返回一个迭代器，然后用迭代器进行遍历。

{% highlight java %}
public Iterator<E> iterator() {  
    return new Itr();  
}
{% endhighlight %}

iterator()方法放回了一个Itr类实例，这个Itr类是ArrayList的一个内部类，实现了Iterator接口。

	int cursor;       // index of next element to return  
	int lastRet = -1; // index of last element returned; -1 if no such  
	int expectedModCount = modCount;

上面是Itr类的三个成员变量。 cursor类似一个游标，指向迭代器下一个值的位置。lastRet是迭代器最后一次取出的元素的位置。expectedModCount的值为Itr初始化时候ArrayList的modCount的值，前面将过modCount用于记录ArrayList内发生结构性改变的次数，而Itr每次进行next或remove的时候都会去检查expectedModCount值是否还和现在的modCount值一直，从而保证了迭代器和ArrayList内数据的一致性。实现代码如下：

{% highlight java %}
public E next() {  
    checkForComodification();  
    int i = cursor;  
    if (i >= size)  
        throw new NoSuchElementException();  
    Object[] elementData = ArrayList.this.elementData;  
    if (i >= elementData.length)  
        throw new ConcurrentModificationException();  
    cursor = i + 1;  
    return (E) elementData[lastRet = i];  
}  
  
final void checkForComodification() {  
    if (modCount != expectedModCount)  
        throw new ConcurrentModificationException();  
}
{% endhighlight %}

Itr作为ArrayList的内部类，可以访问所有ArrayList的成员。理解了这一点，itr类的实现就容易理解了。 

还有一个listIterator的迭代器，实现方式类似。

java的集合在平时开发中很常用，能够了解它们的内部实现，对开发带来很大的便利，也减少了不必要的BUG。
