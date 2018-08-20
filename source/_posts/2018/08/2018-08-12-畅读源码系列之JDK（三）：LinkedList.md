---
title: 畅读源码系列之JDK（三）：LinkedList（未完成...)
comments: true
categories:
  - Source Code Reading - JDK
tags:
  - 畅读源码
abbrlink: 22d500e4
date: 2018-08-12 16:11:00
---
【引言】前一篇说完了ArrayList，这一篇就来说说和ArrayList形影不离，形同姊妹的另外一种结构：LinkedList！
<div align=center><img src="/img/2018/2018-08-12-01.jpg" width="500"/></div>
<!-- more -->

# 类基本定义

## LinkedList
```java
/**
 * 【CHENG】：- 继承关系：LinkedList->AbstractSequentialList->AbstractList->AbstractCollection->Collection
 *            - 其他一些可clone、可序列化的特性不提也罢
 *            - 与ArrayList相比较的话，这里多了个Deque（属于Queue的一个子接口）
 * @since   1.2
 */
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
}
```

# 属性和构造方法

## 基本属性
```java
    /**
     * 【CHENG】：集合的大小（默认是0）
     */
    transient int size = 0;

    /**
	 * 【CHENG】：很明显的链表头（关于Node的结构，猜也能猜到至少包含两个前后指向的指针+一个数据属性）
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
	 * 【CHENG】：很明显的链表尾
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

## 构造方法
```java
    /**
     * 【CHENG】：平时我们最常用的一种构造方法，不指定大小
     * Constructs an empty list.
     */
    public LinkedList() {
    }

    /**
     * 【CHENG】：当然也可以基于其他集合来创建（这里直接就使用了addAll来添加元素完成的）
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

# 核心内部类Node
```java
    /**
     * 【CHENG】：结构其实很简单（真的和根据基本属性推断的结果一样提供了3个元素）
     */
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

# 其他常用方法

## add
```java
    /**
     * ...
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
	
	/**
     * 【CHENG】：新生成一个Node，然后如果集合目前为空，则赋值给第一个；如果已经有链表了，追加到尾部
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
		// 【CHENG】：无处不在的modCount
        modCount++;
    }
	
	/**
     * 【CHENG】：还有个linkBefore方法，是往链表中间插入元素的，流程和前面差不多
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
	
	/**
     * 【CHENG】：至此很容易联想到还有个linkFirst方法是干什么的了
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```

## clear
```java
	/**
     * 【CHENG】：流程本身很简单，就是逐个Node清除（=null）；这里很多地方都有这种=null来help gc的操作，可以借鉴
     */
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```

## indexOf
```java
	/**
     * 【CHENG】：查询索引值的方法，比如contains就会用到这个方法
     */
    public int indexOf(Object o) {
        int index = 0;
		// 【CHENG】：这里又一次出现null和非null用==和equals的区别
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

## offer
```java
	/**
     * 【CHENG】：新生成一个Node，然后如果集合目前为空，则赋值给第一个；如果已经有链表了，追加到尾部
     */
    public boolean offer(E e) {
        return add(e);
    }
```

## add
```java
	/**
     * 【CHENG】：新生成一个Node，然后如果集合目前为空，则赋值给第一个；如果已经有链表了，追加到尾部
     */
```

## add
```java
	/**
     * 【CHENG】：新生成一个Node，然后如果集合目前为空，则赋值给第一个；如果已经有链表了，追加到尾部
     */
```

## add
```java
	/**
     * 【CHENG】：新生成一个Node，然后如果集合目前为空，则赋值给第一个；如果已经有链表了，追加到尾部
     */
```