
---
title: "HashMap的负载因子"
date: 2018-1-22 09:56
categories: 数据结构
tags: 
		- Java
		- HashMap
---

**学习数据结构的笔记**
<!-- more -->

- 在使用LinkedHashMap时,如下:

```java
public class test {
    public static void main(String[] args) {
        Map<String, Object> map = new LinkedHashMap<String, Object>();
    }
}
```

发现LinkedHashMap中的构造方法

```java
public LinkedHashMap() {
        super();
        accessOrder = false;
    }
```
发现调用的父类HashMap其中的一个构造方法

```java
public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

其中的`DEFAULT_LOAD_FACTOR`常量被设定为0.75,如下

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

这就是HashMap负载因子的默认值

- 而HashMap的另一个构造函数:

```java
/** 
 * Constructs an empty <tt>HashMap</tt> with the specified initial 
 * capacity and load factor. 
 * 
 * @param  initialCapacity the initial capacity 
 * @param  loadFactor      the load factor 
 * @throws IllegalArgumentException if the initial capacity is negative 
 *         or the load factor is nonpositive 
 */  
public HashMap(int initialCapacity, float loadFactor) {  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);  
}  
```

- 其中有两个参数`initialCapacity`和`loadFactor`
- `initialCapacity`表示map的初始化容量，initialCapacity > MAXIMUM_CAPACITY，表明map的最大容量是1<<30,也就是1左移30位，每左移一位乘以2，所以就是1*2^30=1073741824
- `loadFactor`是map的负载因子,loadFactor <= 0 || Float.isNaN(loadFactor),表明负载因子要大于0，且是非无穷大的数字

- **负载因子为什么会影响HashMap的性能**?

这里借用[sxlzs_博主的解释](http://blog.csdn.net/sxlzs_/article/details/78688853)

我们都知道有序数组存储数据，对数据的索引效率都很高，但是插入和删除就会有性能瓶颈（回忆`ArrayList`），

链表存储数据，要一次比较元素来检索出数据，所以索引效率低，但是插入和删除效率高（回忆`LinkedList`），

两者取长补短就产生了哈希散列这种存储方式，也就是`HashMap`的存储逻辑.

而负载因子表示一个散列表的空间的使用程度，有这样一个公式：`initailCapacity*loadFactor=HashMap`的容量。

所以负载因子越大则散列表的装填程度越高，也就是能容纳更多的元素，元素多了，链表大了，所以此时索引效率就会降低。

反之，负载因子越小则链表中的数据量就越稀疏，此时会对空间造成烂费，但是此时索引效率高。

- 如何科学的设置`initailCapacity`,`loadFactor`的值:

`HashMap`有三个构造函数，可以选用无参构造函数，不进行设置。默认值分别是16和0.75

官方的建议是`initailCapacity`设置成2的n次幂，`laodFactor`根据业务需求，如果迭代性能不是很重要，可以设置大一下。
