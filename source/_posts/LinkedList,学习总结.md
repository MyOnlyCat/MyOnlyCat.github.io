---
title: 'LinkedList,学习总结'
categories: 数据结构
tags:
  - LinkedList
abbrlink: b0c11e4a
date: 2018-01-22 10:46:07
---

**学习链表结构的记录而已**
<!-- more -->

### 前言

- **队列、堆栈与数组、链表的关系与区分**

- 1.先要搞明白两个东西,数据结构,和数据存储结构

`数据结构`：是指相互之间存在一种或多种特定关系的数据元素的 集合。听起来是不是很抽象，简单理解：数据结构就是描述对象间逻辑关系的学科。比如：队列就是一种先进先出的逻辑结构，栈是一种先进后出的逻辑结构，家谱 是一种树形的逻辑结构！（初学数据结构的时候很不理解为什么有“栈”这个东西；队列很容易理解---无论购物就餐都需要排队；栈可以认为就是个栈道--- 只允许一个人通过的小道，而且只能从一端进入，然后再从这端返回，比如你推了个箱子进去啦，第二个人也推个箱子进去，此时只能等后进来的这个人拉着箱子出 去后，你才能退出。）

`数据存储结构`：它是计算机的一个概念，简单讲，就是描述数据在计算机中存储方式的学科；常用的数据存储 方式就两种：顺序存储，非顺序存储！顺序存储就是把数据存储在一块连续的存储介质（比如硬盘或内存）上----举个例子：从内存中拿出第100个字节到 1000个字节间的连续位置，存储数据；数组就是典型的顺序存储！非顺序存储就是各个数据不一定存在一个连续的位置上，只要每个数据知道它前面的数据和后 面的数据，就能把所有数据连续起来啦；链表就是典型的非顺序存储啦！

队列、栈是线性数据结构的典型代表，而数组、链表是常用的两种数据存储结构；队列和栈均可以用数组或链表的存储方式实现它的功能！

### 双向链表

- `LinkedList`它的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点。这种叫做**双向链表**也叫**双链表**

### LinkedList

- `LinkedList`中定义了3个属性`first`,`last`,`size`,如下:

```java
transient int size = 0;

/**
 * Pointer to first node.
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * Pointer to last node.
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;
```

- `first`表示上一个节点的信息
- `last`表示下一个节点的信息
- `size`表示双向链表中节点实例的个数

### 节点

- `LinkedList`是采用节点`Node`方式把前后两个节点关联起来,变成链表,在`LinkedList`的源码中有个节点的内部类

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

- `LinkedList`之所以是双向链表就是因为`Node`节点中的`next`和`prev`两个参数把链串联起来

    - `next`:保存下一个节点的信息
    - `prev`:保存上一个节点的信息
    - `element`:节点的具体内容(相当于业务数据)
    
### LinkedList中的remove()方法分析

```java
/**
* 返回泛型
*/
public E remove() {
    //调用removeFirst方法
    return removeFirst();
}
```
- `removeFirst()`

```java
/**
 * 删除并返回此列表中的第一个元素。
 *
 * @return 列表中的第一个元素。
 * @throws 如果这个列表是空的，则抛出NoSuchElementException。
 */
public E removeFirst() {
    //把第一个元素拿出来
    final Node<E> f = first;
    //如果为空就报异常
    if (f == null)
        throw new NoSuchElementException();
    //进入unlinkFirst(f)
    return unlinkFirst(f);
}
```

- `unlinkFirst(f)`

```java
/**
 * 取消链接非空的第一个节点f。
 */
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    //从需要移除的元素中把业务数据拿出来
    final E element = f.item;
    //从需要移除的元素中把下一个节点的信息拿出来
    //现在next节点就变成了
    final Node<E> next = f.next;
    //把需要移除的节点的数据设为空
    f.item = null;
    //把需要移除的元素的下一个节点设置为空,利用GC回收
    f.next = null; // help GC
    //然后把这个链表的第一个设置为next节点
    first = next;
    //判断f节点的下一个节点是否为空,为空的话,这个链表的下个节点也就为空,整个双向链表就为空了
    if (next == null)
        last = null;
    else
    //不为空的话,现在next节点就变成了,整个双向链表的第一个节点,所以next节点的上一个节点就为空了
        next.prev = null;
    //双向链表的长度减少
    size--;
    //次数这个是??暂时没弄明白
    modCount++;
    //返回f节点的业务数据
    return element;
    }
```

### LinkedList中的add(E e)方法分析

```java
/**
 *将指定的元素追加到列表的末尾。
 *该方法相当于#addLast。
 * @param e 要添加到此列表中的元素。
 * @return true
 */
public boolean add(E e) {
    //调用linkLast(e)方法
    linkLast(e);
    return true;
}
```

- linkLast(e)

```java
/**
 * 链接e作为最后一个元素。
 */
void linkLast(E e) {
    //让last最后一个节点赋值给l
    final Node<E> l = last;
    //调用内部类Node节点的构造方法
    final Node<E> newNode = new Node<>(l, e, null);
    //让最后一个节点last的值变为newNode
    last = newNode;
    //如果最后一个节点为空,就让newNode变成第一个节点first
    if (l == null)
        first = newNode;
    else
    //不为空的话,让最后一个节点的下一个节点设置为newNode
        l.next = newNode;
    size++;
    modCount++;
}
```

- 内部类`class Node<E>`:

```java
private static class Node<E> {
    E item; //节点业务数据信息,使用泛型
    Node<E> next; //当前节点的下一个节点信息
    Node<E> prev; //当前节点的上一个节点信息
    
    //在new一个Node时,比如add(e) 方法是保存到双向链表的尾部,肯定需要上一个节点的信息
    //参数信息prev:上一个节点;element:当前new的节点所需要的业务数据;next:下一个节点的信息
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element; //1.把需要的业务数据,添加到节点中的item中
        this.next = next; //2.填写下一个节点信息,可以为null,比如在add()方法中就体现了
        this.prev = prev; //3.因为这是一个新的节点所以需要指定上一个节点
    }
}
```

### LinkedList中的get(int index)方法分析

```java
/**
 * 返回列表中指定位置的元素。
 *
 * @param index 返回元素的索引。
 * @return the 元素在列表中的指定位置。
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    //第一步调用checkElementIndex(index)方法检查下标是否越界
    checkElementIndex(index);
    //第二步,调用node(index)方法
    return node(index).item;
}
```

- 第一步检查下标越界情况

    - `checkElementIndex(index)`方法,其中有个构造异常信息的方法
    
    ```java
    /**
     * Constructs an IndexOutOfBoundsException detail message.
     * Of the many possible refactorings of the error handling code,
     * this "outlining" performs best with both server and client VMs.
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }
    
    private void checkElementIndex(int index) {
        //这里调用了isElementIndex(index)方法,判断下标是否越界,越界抛出异常
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    ```
    
    - `isElementIndex(index)`方法
    
    ```java
    /**
     * 说明参数是否为现有元素的索引。
     */
    private boolean isElementIndex(int index) {
        //判断是否下标越界,返回true或者false
        return index >= 0 && index < size;
    }
    ```

- 第二步,确定元素位置
    - `node(index)`方法
    
    ```java
    /**
    * 返回指定元素索引中的(非空)节点。
    */
    Node<E> node(int index) {
    // assert isElementIndex(index);
    
    //这里使用的是二分查找法,如果size右移1位,代表size的一半大小;这种情况下如果,元素的下标index小于的元素的一半说明,元素的位置在前半截,所以从firsy开始遍历
    if (index < (size >> 1)) {
        Node<E> x = first;
        //这里遍历到指定元素的前一位停止,然后取出前一位的下一位,得到指定元素,节约性能
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        //这里同理,从最后一个开始遍历,遍历的下标i-1,如果i大于了,指定元素的下标说明,i这个节点的前一个就是指定的元素,所以用x.prev,就可以了
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
        }
    }
    ```
    
### LinkedList中的set(int index, E element)方法分析

```java
/**
 * 将元素替换为该列表中指定位置的元素。
 *
 * @param 要替换的元素的索引索引。
 * @param 要存储在指定的位置和元素。
 * @return 在指定位置之前的元素的业务数据。
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    //调用检查下标是否越界的方法,和get(Int index)方法一样
    checkElementIndex(index);
    //取出指定下标位置的节点node(index)方法和上面一样
    Node<E> x = node(index);
    //把节点x的老数据取出来
    E oldVal = x.item;
    //把新数据给x节点
    x.item = element;
    //返回x节点的老数据
    return oldVal;
}
```
    




