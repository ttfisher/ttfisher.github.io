---
title: 畅读源码系列之JDK（三）：LinkedList
comments: true
categories:
  - 技术结构升级
  - JDK源码汇
tags:
  - 畅读源码
abbrlink: 22d500e4
date: 2018-08-12 16:11:00
---
【引言】前一篇说完了ArrayList，这一篇就来说说和ArrayList形影不离，形同姊妹的另外一种结构：LinkedList！
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-08-12-01.jpg" width="55%"/></div>
<!-- more -->

# 类的定义
```java
/**
 * 【Lin.C】：- 继承关系：LinkedList->AbstractSequentialList->AbstractList->AbstractCollection->Collection
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

# 成员变量
```java
    /**
     * 【Lin.C】：集合的大小（默认是0）
     */
    transient int size = 0;

    /**
     * 【Lin.C】：很明显的链表头（关于Node的结构，猜也能猜到至少包含两个前后指向的指针+一个数据属性）
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * 【Lin.C】：很明显的链表尾
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

# 构造方法
```java
    /**
     * 【Lin.C】：平时我们最常用的一种构造方法，不指定大小
     * Constructs an empty list.
     */
    public LinkedList() {
    }

    /**
     * 【Lin.C】：当然也可以基于其他集合来创建（这里直接就使用了addAll来添加元素完成的）
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

# 增

## add
```java
    /**
     * 【Lin.C】：将对象e插入队列尾部，成功返回true，失败（没有空间）返回false（来自于Collection接口）
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    /**
     * 【Lin.C】：新生成一个Node，然后如果集合目前为空，则赋值给第一个；如果已经有链表了，追加到尾部
     */
    void linkLast(E e) {
        final Node<E> l = last;
        
        // 【Lin.C】：生成新节点
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        
        // 【Lin.C】：最后一个元素是不是为空？（为空实际就意味着该链表是空的）
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        
        // 【Lin.C】：记一下大小变化
        size++;
        
        // 【Lin.C】：无处不在的modCount
        modCount++;
    }
    
    /**
     * 【Lin.C】：还有个linkBefore方法，是往链表某个元素的前面插入一个新元素的，流程和前面差不多
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
     * 【Lin.C】：至此很容易联想到还有个linkFirst方法是干什么的了
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

## offer
```java
    /**
     * 【Lin.C】：将对象e插入队列尾部，成功返回true，失败（没有空间）返回false（来自于Queue接口）
     */
    public boolean offer(E e) {
        return add(e);
    }
```

# 删

## remove
```java
    /**
     * 【Lin.C】：获取并移除队列头部元素，如果队列为空，抛出异常（来自于Queue接口）
     * 【注】：和poll的区别在于空队列时的返回方式不同
     */
    public E remove() {
        return removeFirst();
    }
    
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f); // 参考前一节
    }
```

## remove
```java
    /**
     * 【Lin.C】：来自于AbstractSequentialList的移除方法，移除某一个index的元素
     */
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
    
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

## clear
```java
    /**
     * 【Lin.C】：流程本身很简单，就是逐个Node清除（=null）；这里很多地方都有这种=null来help gc的操作，可以借鉴
     */
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        // 【Lin.C】：这里用了个临时变量来从头至尾置空，为什么不从尾至头呢？
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

# 改

## set
```java
    /**
     * 【Lin.C】：将某个位置的元素值修改掉
     */
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```

# 查

## peek
```java
    /**
     * 【Lin.C】：获取但不移除队列头部元素，如果队列为空，返回null（也是来自于Queue接口）
     
     */
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```

## poll
```java
    /**
     * 【Lin.C】：获取并移除队列头部元素，如果队列为空，返回null（来自于Queue接口）
     */
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
    
    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

## indexOf
```java
    /**
     * 【Lin.C】：查询索引值的方法，比如contains就会用到这个方法
     */
    public int indexOf(Object o) {
        int index = 0;
        // 【Lin.C】：这里又一次出现null和非null用==和equals的区别
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

# 内部类

## Node
```java
    /**
     * 【Lin.C】：结构其实很简单（真的和根据基本属性推断的结果一样提供了3个元素）
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
        ......
    }
```