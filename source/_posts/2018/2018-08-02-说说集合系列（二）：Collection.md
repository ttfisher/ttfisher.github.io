---
title: 说说集合系列（二）：Collection
comments: true
categories:
  - JDK one by one - collection plus
tags:
  - Java语言特性
  - Java集合
abbrlink: f29d3a59
date: 2018-08-02 14:07:00
---
【引言】接着上一篇，在纲领性的说明完成后，从本篇开始，按照不同的类型和特性，逐步的对不同的集合类型进行一一的分析和解读，从细节上把控所有的集合特性。
<div align=center><img src="/img/2018-08-02-10.jpg" width="500"/></div>
<!-- more -->

# Collection的定义
&emsp;&emsp;通过下图可以很清晰的看到：（1）Collection是继承Iterable接口的；（2）Iterable提供了迭代器获取的方法；（3）Collection内提供了基本的增删查等常用操作；
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-02-11.jpg" width="60%">

# Collection的分类

## 分类
&emsp;&emsp;Collection是继承了Iterable接口的一个集合顶级接口，在它之下又有3个分支，其中我们较常用的是List和Set，而对于Queue平时使用的不多，也知之甚少，正好可以借着这次整理的机会熟悉一下。
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-02-09.jpg" width="75%">

## 特点
- List：可以重复、有序
- Set：不可重复、可有序可无序
- Queue：先入先出的队列结构

# List

## ArrayList

### 接口/类定义
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    ......
    /**
     * Default initial capacity. [默认初始化数组扩容基数]
     */
    private static final int DEFAULT_CAPACITY = 10;

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * Constructs an empty list with the specified initial capacity.
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
    
    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    ......
}
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-01.jpg" width="75%">

### 数据结构扩容
&emsp;&emsp;ArrayList的扩容是在容量已经无法支撑服务的情况下才会触发的
```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：初始化大小是10（如果知道最大容量，可以在初始化的时候就指定大小，能节省扩容的性能开销） 
- 扩容：默认扩容为已有结构长度的1.5倍；最大长度比Integer的最大值少8（一般应该用不到这么大的）
- 构造：只能基于自身的特性构造
- 线程安全性：不安全，因为所有方法都没有进行同步控制（synchronize或lock）
- 有序性：有序

### Example
```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

/**
 * Created by chenglin on 2018/8/2.
 */
public class P {

    /**
     * 欣赏一下几种不同的Collection遍历方式吧
     * @param args
     */
    public static void main(String[] args) {

        // 和Arrays.asList()有区别，具体区别是什么？
        List<String> list = new ArrayList<String>();
        list.add("I");
        list.add("am");
        list.add("chenglin");
        list.add(".");
        list.add("Haha");
        System.out.println("*********************************************");

        // 第一种遍历方式
        list.forEach(value -> {
            System.out.println("[1] Value = " + value);
        });
        System.out.println("*********************************************");

        // 第二种遍历方式
        Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()) {
            System.out.println("[2] Value = " + iterator.next());
        }
        System.out.println("*********************************************");

        // 第三种遍历方式
        Iterator<String> iterator2 = list.iterator();
        iterator2.forEachRemaining(value -> {
            System.out.println("[3] Value = " + value);
        });
        System.out.println("*********************************************");

        // 删除一个元素
        System.out.println("Origin size = " + list.size());
        Iterator<String> iterator3 = list.iterator();
        while (iterator3.hasNext()) {
            String value = iterator3.next();
            System.out.println("[4] Value = " + value);
            if(value.equals("Haha")){
                iterator3.remove();
            }
        }
        System.out.println("Remaining size = " + list.size());
        System.out.println("*********************************************");

    }
}
```

## LinkedList

### 接口/类定义
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    ......
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

    /**
     * Constructs an empty list.
     */
    public LinkedList() {
    }
    
    // 节点结构：当前对象，前一个节点的引用，后一个节点的引用
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
    ......
}
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-02.jpg" width="75%">

### 数据结构扩容
**&emsp;&emsp;因为LinkedList本身是以链表作为存储结构的，理论上在不超过内存限制和链表本身限制的情况下，是没有容量大小的说法的，所以也谈不上扩容**

### 特性微总结
- 底层存储结构：双向链表（特性是：读慢，写快；也是相对于数组结构而言的）
- 初始化大小：不涉及
- 扩容：不涉及（理论上可以随意追加）
- 构造：只能基于自身的特性构造
- 线程安全性：不安全，因为所有方法都没有进行同步控制（synchronize或lock）
- 有序性：有序

## Vector

### 接口/类定义
```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    /**
     * The array buffer into which the components of the vector are
     * stored. The capacity of the vector is the length of this array buffer,
     * and is at least large enough to contain all the vector's elements.
     */
    protected Object[] elementData;

    /**
     * Constructs an empty vector with the specified initial capacity and
     * capacity increment.
     */
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }

    /**
     * Constructs an empty vector with the specified initial capacity and
     * with its capacity increment equal to zero.
     */
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }

    /**
     * Constructs an empty vector so that its internal data array
     * has size {@code 10} and its standard capacity increment is
     * zero.
     */
    public Vector() {
        this(10);
    }

    /**
     * Constructs a vector containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     */
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
    
    // A synchronized function show
    public synchronized void copyInto(Object[] anArray) {
        System.arraycopy(elementData, 0, anArray, 0, elementCount);
    }
}
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-03.jpg" width="75%">

### 数据结构扩容
```java
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：初始化大小默认为10（如果知道最大容量，可以在初始化的时候就指定大小，能节省扩容的性能开销） 
- 扩容：若指定了capacityIncrement，则按照这个值扩容；若未指定，默认每次基于原大小（oldCapacity）翻倍（这里默认情况下是比ArrayList扩容大小要大50%的）
- 构造：除了基于自身的特性构造，还可以由其它Collection转换（基于toArray转换为数组后，通过数组copy实现）
- 线程安全性：安全，因为必要的方法都有加synchronize进行了同步控制
- 有序性：有序

## Stack

### 接口/类定义
```java
public class Stack<E> extends Vector<E> {
    /**
     * Creates an empty Stack.
     */
    public Stack() {
    }
}
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-04.jpg" width="75%">

### 数据结构扩容
```java
// 继承自Vector
```

### 特性微总结
- **重点：Stack继承自Vector，在其基础之上添加了一些入栈，出栈的操作，如push()/pop()/peek()等；而且这些操作也是synchronize的。**
- 底层存储结构：同Vector
- 初始化大小：同Vector
- 扩容：同Vector
- 构造：同Vector
- 线程安全性：同Vector
- 有序性：有序
- 其他：通常可以利用Deque或者Deque的实现类来替换Stack，比如ArrayDeque可以用来作为栈或者队列（不过它非线程安全的，且不支持null元素）

## CopyOnWriteArrayList

### 接口/类定义
```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {

    /** The lock protecting all mutators */
    final transient ReentrantLock lock = new ReentrantLock();

    /** The array, accessed only via getArray/setArray. */
    private transient volatile Object[] array;
    
    /**
     * Creates an empty list.
     */
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }

    /**
    * Creates a list containing the elements of the specified
    * collection, in the order they are returned by the collection's
    * iterator.
    *
    * @param c the collection of initially held elements
    * @throws NullPointerException if the specified collection is null
    */
    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }
}
```

### 线程安全和COW
&emsp;&emsp;该类有用到ReentrantLock来实现线程安全，比如它的set方法实现如下，有显示的加锁和解锁操作；
&emsp;&emsp;set方法内部有一次很明显的复制和替换的操作（16-18行），这个操作的过程就是通常称为COW的操作；（可以简单理解为读写分离锁，不影响读操作，具体这个概念后面计划专门写一篇文章来细说）

```java
    /**
     * Replaces the element at the specified position in this list with the
     * specified element.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);

            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-05.jpg" width="75%">

### 数据结构扩容
&emsp;&emsp;实际上CopyOnWriteArrayList每完成一次数据的写入或删除，都涉及到数组大小的调整，而不是像ArrayList那样达到固定大小才做扩容，可以理解为CopyOnWriteArrayList每次只扩容一个元素
```java
   public void add(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            if (index > len || index < 0)
                throw new IndexOutOfBoundsException("Index: "+index+
                                                    ", Size: "+len);
            Object[] newElements;
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(elements, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index, newElements, index + 1,
                                 numMoved);
            }
            newElements[index] = element;
            setArray(newElements);
        } finally {
            lock.unlock();
        }
    }
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：初始化大小默认为0
- 扩容：每次有数据出入都会引发一次COW操作（也就是扩容操作，每次一个元素）
- 构造：除了基于自身的特性构造，还可以由其它Collection转换（基于toArray转换为数组后，通过数组copy实现）
- 线程安全性：安全，这里不是使用synchronize来做的，而是通过ReentrantLock来实现的
- 有序性：有序

# Set

## HashSet

### 接口/类定义
&emsp;&emsp;如果不看源码，可能真的想不到HashSet是通过HashMap来实现的；HashMap天然是key去重的，而这个key对应的value都是一个final的常量（具体是什么并不重要）
```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{

    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * default initial capacity (16) and load factor (0.75).
     */
    public HashSet() {
        map = new HashMap<>();
    }

    /**
     * Constructs a new set containing the elements in the specified
     * collection.  The <tt>HashMap</tt> is created with default load factor
     * (0.75) and an initial capacity sufficient to contain the elements in
     * the specified collection.
     */
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * the specified initial capacity and the specified load factor.
     */
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * the specified initial capacity and default load factor (0.75).
     */
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    /**
     * Constructs a new, empty linked hash set.  (This package private
     * constructor is only used by LinkedHashSet.) The backing
     * HashMap instance is a LinkedHashMap with the specified initial
     * capacity and the specified load factor.
     */
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
}
```

### 基于Collection初始化
&emsp;&emsp;实际还是先扩容，然后逐个调用add方法将原Collection里面的元素塞到HashMap中
```java
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```


### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-06.jpg" width="75%">

### 数据结构扩容
&emsp;&emsp;实际上也不涉及独立的扩容操作，底层只是往HashMap里面put一个新值而已（具体HashMap怎么扩容的，就在下一篇文章见分晓吧！）
```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

### 特性微总结
- **底层存储结构：HashMap（令人震惊的发现）**
- 初始化大小：空的HaspMap
- 扩容：参考HashMap的扩容
- 构造：除了基于自身的特性构造，还可以由其它Collection转换
- 线程安全性：不安全，没有任何的同步和锁操作
- 有序性：无序

## TreeSet

### 接口/类定义
&emsp;&emsp;意料之中，还是通过一个Map来作为底层结构的，只是这次换了一种类型NavigableMap（这个接口是继承自SortedMap的，所以肯定是有序的，这个纯粹是根据类名猜测的，事实证明这个大胆的猜测是正确的）
```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{
    /**
     * The backing map.
     */
    private transient NavigableMap<E,Object> m;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    /**
     * Constructs a set backed by the specified navigable map.
     */
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }

    /**
     * Constructs a new, empty tree set, sorted according to the
     * natural ordering of its elements.  
     */
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }

    /**
     * Constructs a new, empty tree set, sorted according to the specified
     * comparator. 
     */
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }

    /**
     * Constructs a new tree set containing the elements in the specified
     * collection, sorted according to the <i>natural ordering</i> of its
     * elements.  
     */
    public TreeSet(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    /**
     * Constructs a new tree set containing the same elements and
     * using the same ordering as the specified sorted set.
     */
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }
}
```

### 基于Comparator初始化
&emsp;&emsp;基于其他Collection的初始化其实都大同小异，这里比较特殊的是用到了基于Comparator的初始化（这个Comparator目测是用于排序的），所以看看这个初始化的过程是如何实现的
```java
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }

    // 生成一个Map（TreeMap本身是implements NavigableMap<K,V>的）
    public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
    
    // 给this也就是TreeSet一个赋值（这个值就是前面生成的那个Map）
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-07.jpg" width="75%">

### 数据结构扩容
```java
// 可参考HashSet的实现方式，雷同，最终都是归结到HashMap的扩容上
```

### 特性微总结
- **底层存储结构：NavigableMap（不那么震惊了）**
- 初始化大小：空的NavigableMap
- 扩容：参考NavigableMap的扩容
- 构造：除了基于自身的特性构造，还可以由其它Collection、基于Comparator初始化
- 线程安全性：不安全，没有任何的同步和锁操作
- 有序性：有序

## LinkedHashSet

### 接口/类定义
&emsp;&emsp;LinkedHashSet继承自HashSet，源码更少、更简单，唯一的区别是LinkedHashSet内部使用的是LinkHashMap（通过super升级到父类HashSet里面实现的）。这样做的意义或者好处就是LinkedHashSet中的元素顺序是可以保证的，也就是说遍历序和插入序是一致的。
```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {

    /**
     * Constructs a new, empty linked hash set with the specified initial
     * capacity and load factor.
     */
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }

    /**
     * Constructs a new, empty linked hash set with the specified initial
     * capacity and the default load factor (0.75).
     */
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    /**
     * Constructs a new, empty linked hash set with the default initial
     * capacity (16) and load factor (0.75).
     */
    public LinkedHashSet() {
        super(16, .75f, true);
    }

    /**
     * Constructs a new linked hash set with the same elements as the
     * specified collection.  
     */
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
}

// 上面的所有构造方法，基本都是通过super调用了HashSet的这个构造方法实现的（注：这个方法是default的，所以对外是不可见的）
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
   
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
}

```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-08.jpg" width="75%">

### 数据结构扩容
```java
// 可参考HashSet的实现方式，雷同，最终都是归结到HashMap的扩容上
```

### 特性微总结
- 底层存储结构：LinkedHashMap
- 初始化大小：空的LinkedHashMap
- 扩容：参考LinkedHashMap的扩容
- 构造：基于自身的特性构造，可以通过多种不同的构造方法实现
- 线程安全性：不安全，没有任何的同步和锁操作
- 有序性：有序（因为LinkedHashMap是有序的）

## CopyOnWriteArraySet

### 接口/类定义
&emsp;&emsp;真的是天下之大无奇不有啊，CopyOnWriteArraySet的底层数据结构居然是CopyOnWriteArrayList
```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }

    /**
     * Creates a set containing all of the elements of the specified
     * collection.
     */
    public CopyOnWriteArraySet(Collection<? extends E> c) {
        if (c.getClass() == CopyOnWriteArraySet.class) {
            @SuppressWarnings("unchecked") CopyOnWriteArraySet<E> cc =
                (CopyOnWriteArraySet<E>)c;
            al = new CopyOnWriteArrayList<E>(cc.al);
        }
        else {
            al = new CopyOnWriteArrayList<E>();
            al.addAllAbsent(c);
        }
    }
}
```

### 线程安全保障
&emsp;&emsp;和CopyOnWriteArrayList类似，也是通过ReentrantLock来实现锁操作，从而确保整个操作的线程安全性。
```java
    /**
     * A version of addIfAbsent using the strong hint that given
     * recent snapshot does not contain e.
     */
    private boolean addIfAbsent(E e, Object[] snapshot) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] current = getArray();
            int len = current.length;
            if (snapshot != current) {
                // Optimize for lost race to another addXXX operation
                int common = Math.min(snapshot.length, len);
                for (int i = 0; i < common; i++)
                    if (current[i] != snapshot[i] && eq(e, current[i]))
                        return false;
                if (indexOf(e, current, common, len) >= 0)
                        return false;
            }
            Object[] newElements = Arrays.copyOf(current, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

### 有序性验证
&emsp;&emsp;对于有序性，仅靠感觉没有多大把握（虽然通过底层实现是CopyOnWriteArrayList基本可以猜出是有序的），所以动手操作一下才是最稳妥的，事实也证明前面的猜测是正确的。
```java
[HashSet]：
    public static void main(String[] args) {
        HashSet set = new HashSet();
        set.add("I");
        set.add("am");
        set.add("from");
        set.add("jiangning");
        set.add("nanjing");

        set.forEach(value->{
            System.out.println("Value is :" + value);
        });
    }
[结果]：
    Value is :nanjing
    Value is :I
    Value is :from
    Value is :am
    Value is :jiangning
    
[CopyOnWriteArraySet]：
    public static void main(String[] args) {
        CopyOnWriteArraySet set = new CopyOnWriteArraySet();
        set.add("I");
        set.add("am");
        set.add("from");
        set.add("jiangning");
        set.add("nanjing");

        set.forEach(value->{
            System.out.println("Value is :" + value);
        });
    }
[结果]：
    Value is :I
    Value is :am
    Value is :from
    Value is :jiangning
    Value is :nanjing
```
### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-09.jpg" width="50%">

### 数据结构扩容
```java
    /**
     * A version of addIfAbsent using the strong hint that given
     * recent snapshot does not contain e.
     */
    private boolean addIfAbsent(E e, Object[] snapshot) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] current = getArray();
            int len = current.length;
            if (snapshot != current) {
                // Optimize for lost race to another addXXX operation
                int common = Math.min(snapshot.length, len);
                for (int i = 0; i < common; i++)
                    if (current[i] != snapshot[i] && eq(e, current[i]))
                        return false;
                if (indexOf(e, current, common, len) >= 0)
                        return false;
            }
            
            // 扩容
            Object[] newElements = Arrays.copyOf(current, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：初始化大小默认为0
- 扩容：每次有数据出入都会引发一次COW操作（也就是扩容操作，每次一个元素）
- 构造：基于自身的特性构造
- 线程安全性：安全，这里不是使用synchronize来做的，而是通过ReentrantLock来实现的
- 有序性：有序（CopyOnWriteArrayList的关系）

# Queue
> Queue 也是 Java 集合框架中定义的一种接口，直接继承自 Collection 接口。除了基本的 Collection 接口规定测操作外，Queue 接口还定义一组针对队列的特殊操作。通常来说，Queue 是按照先进先出(FIFO)的方式来管理其中的元素的，但是优先队列是一个例外。

## AbstractQueue

### 接口/类定义
```java
public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> {

    /**
     * Constructor for use by subclasses.
     */
    protected AbstractQueue() {
    }
}
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-10.jpg" width="40%">

### 数据结构扩容
```java
// 暂不涉及；实际使用时是通过Queue接口的boolean offer(E e);方法实现的add
```

### 特性微总结
- AbstractQueue本身是一个抽象类，实际应用时不会使用到此类型，所以这里不涉及具体特性总结，仅供其他结构参考

## PriorityQueue

### 接口/类定义
&emsp;&emsp;PriorityQueue的构造方法很多，可通过不同维度来构造；实际上这个Queue的底层还是一个数组来支撑的，默认初始化容量是11（至于为什么是11也没找到具体的说法）
```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {

    private static final int DEFAULT_INITIAL_CAPACITY = 11;

    /**
     * Priority queue represented as a balanced binary heap: the two
     * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
     * priority queue is ordered by comparator, or by the elements'
     * natural ordering, if comparator is null: For each node n in the
     * heap and each descendant d of n, n <= d.  The element with the
     * lowest value is in queue[0], assuming the queue is nonempty.
     */
    transient Object[] queue; // non-private to simplify nested class access
    
    /**
     * Creates a {@code PriorityQueue} with the default initial
     * capacity (11) that orders its elements according to their
     * {@linkplain Comparable natural ordering}.
     */
    public PriorityQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
    }

    /**
     * Creates a {@code PriorityQueue} with the specified initial
     * capacity that orders its elements according to their
     * {@linkplain Comparable natural ordering}.
     */
    public PriorityQueue(int initialCapacity) {
        this(initialCapacity, null);
    }

    /**
     * Creates a {@code PriorityQueue} with the default initial capacity and
     * whose elements are ordered according to the specified comparator.
     * @since 1.8
     */
    public PriorityQueue(Comparator<? super E> comparator) {
        this(DEFAULT_INITIAL_CAPACITY, comparator);
    }

    /**
     * Creates a {@code PriorityQueue} with the specified initial capacity
     * that orders its elements according to the specified comparator.
     */
    public PriorityQueue(int initialCapacity,
                         Comparator<? super E> comparator) {
        // Note: This restriction of at least one is not actually needed,
        // but continues for 1.5 compatibility
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
    }

    /**
     * Creates a {@code PriorityQueue} containing the elements in the
     * specified collection. 
     */
    @SuppressWarnings("unchecked")
    public PriorityQueue(Collection<? extends E> c) {
        if (c instanceof SortedSet<?>) {
            SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
            this.comparator = (Comparator<? super E>) ss.comparator();
            initElementsFromCollection(ss);
        }
        else if (c instanceof PriorityQueue<?>) {
            PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
            this.comparator = (Comparator<? super E>) pq.comparator();
            initFromPriorityQueue(pq);
        }
        else {
            this.comparator = null;
            initFromCollection(c);
        }
    }

    /**
     * Creates a {@code PriorityQueue} containing the elements in the
     * specified priority queue. 
     */
    @SuppressWarnings("unchecked")
    public PriorityQueue(PriorityQueue<? extends E> c) {
        this.comparator = (Comparator<? super E>) c.comparator();
        initFromPriorityQueue(c);
    }

    /**
     * Creates a {@code PriorityQueue} containing the elements in the
     * specified sorted set.   
     */
    @SuppressWarnings("unchecked")
    public PriorityQueue(SortedSet<? extends E> c) {
        this.comparator = (Comparator<? super E>) c.comparator();
        initElementsFromCollection(c);
    }
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-11.jpg" width="45%">

### 数据结构扩容
```java
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity of the array.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50% ****
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
        // overflow-conscious code
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        queue = Arrays.copyOf(queue, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：初始化大小是11（如果知道最大容量，可以在初始化的时候就指定大小，能节省扩容的性能开销） 
- 扩容：newCapacity = oldCapacity + ((oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1));
- 构造：很多构造方法
- 线程安全性：不安全，因为所有方法都没有进行同步控制（synchronize或lock）
- 有序性：有序（但是被判定为相等的元素，通过poll方法依次取出它们时,它们的顺序是不确定的）

## BlockingQueue && ArrayBlockingQueue

### 接口/类定义
&emsp;&emsp;一个由数组结构组成的有界阻塞队列，首先底层结构是数组，另外这个数组本身是有界的（大小固定的）;默认使用的是非公平锁来实现线程安全控制的
```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {

    /**
     * Serialization ID. This class relies on default serialization
     * even for the items array, which is default-serialized, even if
     * it is empty. Otherwise it could not be declared final, which is
     * necessary here.
     */
    private static final long serialVersionUID = -817911632652898426L;

    /** The queued items */
    final Object[] items;

    /**
     * Creates an {@code ArrayBlockingQueue} with the given (fixed)
     * capacity and default access policy.
     */
    public ArrayBlockingQueue(int capacity) {
        this(capacity, false);
    }

    /**
     * Creates an {@code ArrayBlockingQueue} with the given (fixed)
     * capacity and the specified access policy.
     */
    public ArrayBlockingQueue(int capacity, boolean fair) {
        if (capacity <= 0)
            throw new IllegalArgumentException();
        this.items = new Object[capacity];
        lock = new ReentrantLock(fair);
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }

    /**
     * Creates an {@code ArrayBlockingQueue} with the given (fixed)
     * capacity, the specified access policy and initially containing the
     * elements of the given collection,
     * added in traversal order of the collection's iterator.
     */
    public ArrayBlockingQueue(int capacity, boolean fair,
                              Collection<? extends E> c) {
        this(capacity, fair);

        final ReentrantLock lock = this.lock;
        lock.lock(); // Lock only for visibility, not mutual exclusion
        try {
            int i = 0;
            try {
                for (E e : c) {
                    checkNotNull(e);
                    items[i++] = e;
                }
            } catch (ArrayIndexOutOfBoundsException ex) {
                throw new IllegalArgumentException();
            }
            count = i;
            putIndex = (i == capacity) ? 0 : i;
        } finally {
            lock.unlock();
        }
    }

}
```

### 关于阻塞
&emsp;&emsp;所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦条件满足，被挂起的线程又会自动被唤醒；当队列中没有数据的情况下，消费者端的所有线程都会被自动阻塞（挂起），直到有数据放入队列，当队列中填满数据的情况下，生产者端的所有线程都会被自动阻塞（挂起），直到队列中有空的位置，线程被自动唤醒。

### 关于有界
```java
[Main]
    public static void main(String[] args) {
        ArrayBlockingQueue queue = new ArrayBlockingQueue(3);
        queue.add("1");
        System.out.println("queue.remainingCapacity() = " + queue.remainingCapacity());
        queue.add("2");
        System.out.println("queue.remainingCapacity() = " + queue.remainingCapacity());
        queue.add("3");
        System.out.println("queue.remainingCapacity() = " + queue.remainingCapacity());

        queue.forEach(value->{
            System.out.println("Value is : " + value);
        });
    }

[结果]:
    queue.remainingCapacity() = 2
    queue.remainingCapacity() = 1
    queue.remainingCapacity() = 0
    Value is : 1
    Value is : 2
    Value is : 3
    
[越界]:
    Exception in thread "main" java.lang.IllegalStateException: Queue full
        at java.util.AbstractQueue.add(AbstractQueue.java:98)
        at java.util.concurrent.ArrayBlockingQueue.add(ArrayBlockingQueue.java:312)
        at com.xunsiya.P.main(P.java:30)
```
### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-12.jpg" width="60%">

### 数据结构扩容
```java
// 有界的，不涉及扩容
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：初始化大小由创建者通过构造函数指定 
- 扩容：不涉及
- 构造：很多构造方法（可指定初始化容量、锁的类型等）
- 线程安全性：安全，用到了ReentrantLock
- 有序性：有序

### 类BlockingQueue
- ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。 
- LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。 
- PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。 
- DealyQueue：一个使用优先级队列实现的无界阻塞队列。 
- SynchronousQueue：一个不存储元素的阻塞队列。 
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。 
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。（摘自《Java并发编程的艺术》） 

## Deque & ArrayDeque

### 接口/类定义
&emsp;&emsp;Deque 接口继承自 Queue接口，但 Deque 支持同时从两端添加或移除元素，因此又被成为双端队列。鉴于此，Deque 接口的实现可以被当作 FIFO队列使用，也可以当作LIFO队列（栈）来使用，官方也是推荐使用 Deque 的实现来替代 Stack。
&emsp;&emsp;ArrayDeque 是 Deque 接口的一种具体实现，是依赖于可变数组来实现的。ArrayDeque 没有容量限制，可根据需求自动进行扩容。ArrayDeque不支持值为 null 的元素。
```java
public class ArrayDeque<E> extends AbstractCollection<E>
                           implements Deque<E>, Cloneable, Serializable
{
    /**
     * The array in which the elements of the deque are stored.
     */
    transient Object[] elements; // non-private to simplify nested class access

    /**
     * Constructs an empty array deque with an initial capacity
     * sufficient to hold 16 elements.
     */
    public ArrayDeque() {
        elements = new Object[16];
    }

    /**
     * Constructs an empty array deque with an initial capacity
     * sufficient to hold the specified number of elements.
     */
    public ArrayDeque(int numElements) {
        allocateElements(numElements);
    }

    /**
     * Constructs a deque containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator. 
     */
    public ArrayDeque(Collection<? extends E> c) {
        allocateElements(c.size());
        addAll(c);
    }
}
```

### 内存空间开辟
```java
private void allocateElements(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;
        // Find the best power of two to hold elements.
        // Tests "<=" because arrays aren't kept full.
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++;

            if (initialCapacity < 0)   // Too many elements, must back off
                initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
        }
        elements = new Object[initialCapacity];
    }
```

### 关于有序和扩容
```java
[Main]:
    public static void main(String[] args) {
        ArrayDeque queue = new ArrayDeque(3);
        queue.add("1");
        queue.add("2");
        queue.add("3");
        queue.add("4");
        queue.add("5");

        queue.forEach(value->{
            System.out.println("Value is : " + value);
        });
    }

[结果]:
	Value is : 1
	Value is : 2
	Value is : 3
	Value is : 4
	Value is : 5
```

### 继承关系简图
<img style="clear: both;display: block;margin:auto;" src="/img/2018-08-03-13.jpg" width="50%">

### 数据结构扩容
```java
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }
    
    public boolean add(E e) {
        addLast(e);
        return true;
    }
    
    public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        elements[tail] = e;
        if ( (tail = (tail + 1) & (elements.length - 1)) == head)
            doubleCapacity();
    }

    private void doubleCapacity() {
        assert head == tail;
        int p = head;
        int n = elements.length;
        int r = n - p; // number of elements to the right of p
        int newCapacity = n << 1;
        if (newCapacity < 0)
            throw new IllegalStateException("Sorry, deque too big");
        Object[] a = new Object[newCapacity];
        System.arraycopy(elements, p, a, 0, r);
        System.arraycopy(elements, 0, a, r, p);
        elements = a;
        head = 0;
        tail = n;
    }
```

### 特性微总结
- 底层存储结构：数组（特性是：读快，写慢；也是相对于链表结构而言的）
- 初始化大小：MIN_INITIAL_CAPACITY=8
- 扩容：可进行double扩容（doubleCapacity）
- 构造：可默认、基于容量、基于其他集合构造
- 线程安全性：不安全，不涉及线程安全的控制
- 有序性：有序

### Deque的其他实现
- ArrayDeque (java.util)：数组实现的Deque
- BlockingDeque (java.util.concurrent) ：接口
- ConcurrentLinkedDeque (java.util.concurrent)：链表实现的线程安全的Deque（用到了UNSAFE和CAS，关于这两个概念，后面会专门开一个章节）
- LinkedBlockingDeque (java.util.concurrent)：链表实现的非线程安全的Deque
- LinkedList (java.util)：链表实现的Deque（List）
