---
title: 畅读源码系列之JDK（二）：ArrayList
comments: true
categories:
  - 技术结构升级
  - JDK源码汇
tags:
  - 畅读源码
abbrlink: 773b69e2
date: 2018-08-11 10:11:00
---
【引言】针对集合实际上我们有一个专题有了一个基础的说明，但是还不够深入，所以，借着这个源码畅读系列的整理的机会，在这里好好的读读几个重点集合类的源码，第一个最常接触的集合类就是ArrayList了，所以就从它开始了！
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-08-11-01.jpg" width="55%"/></div>
<!-- more -->

# 类的定义
```java
/**
 * 【Lin.C】：从ArrayList本身的定义出发，可以简单分析出它的基本特性
 *         - 溯源的话，还是回到Collection；继承关系：ArrayList->AbstractList->AbstractCollection->Collection
 *         - 其他一些可clone、可序列化的特性不提也罢
 *         - RandomAccess，和后两个类似的都是标记接口，表示该实现类支持快速随机访问（另立专题讨论）
 * @since   1.2
 */
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
}
```

# 成员变量
```java
    /**
     * 【Lin.C】：字面意思就可以理解：默认容量
     */
    private static final int DEFAULT_CAPACITY = 10;
    
    /**
     * 【Lin.C】：默认的空实例
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    /**
     * 【Lin.C】：实际存储的数组（因为ArrayList底层就是数组结构）
     */
    transient Object[] elementData; // non-private to simplify nested class access
```

# 构造方法

## ArrayList()
```java
    /**
     * 【Lin.C】：平时我们最常用的一种构造方法，不指定大小；默认初始化一个空数组，第一次add就会引发扩容
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

## ArrayList(int initialCapacity)
```java
    /**
     * 【Lin.C】：这个是指定容量的构造方法；通常对于可预知数量的对象存储使用这种方式初始化
     */
    public ArrayList(int initialCapacity) {
        // 【Lin.C】：initialCapacity只能大于等于0，一旦小于0就会抛出异常了
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

## ArrayList(Collection<? extends E> c)
```java
    /**
     * 【Lin.C】：基于别的集合来构造
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // 【Lin.C】：必须是对象数组
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                // 【Lin.C】：然后通过底层的数组拷贝技术（实际对应：System.arraycopy）完成数组的复制
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

# 增

## add
```java
    /**
     * 【Lin.C】：流程很简单
     */
    public boolean add(E e) {
        // 【Lin.C】：先确认是否需要扩容（见下一个方法说明）
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        
        // 【Lin.C】：然后把新的数据追加到数组末尾
        elementData[size++] = e;
        
        // 【Lin.C】：然后直接返回true了
        return true;
    }
    
    /**
     * 【Lin.C】：这个也是个衔接的方法
     */
    private void ensureCapacityInternal(int minCapacity) {
    
        // 【Lin.C】：首先看看初始化数组是不是空的，如果是空的，则初始化一下最小容量
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // 【Lin.C】：最小容量的取值就在默认容量（10）和传进来的参数容量中取大
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        // 【Lin.C】：见下一个方法说明
        ensureExplicitCapacity(minCapacity);
    }

    /**
     * 【Lin.C】：这个也是个衔接的方法
     */
    private void ensureExplicitCapacity(int minCapacity) {
        
        /**
         * 【Lin.C】：- 定义于AbstractList（protected transient int modCount = 0;）
         *           - modCount记录list结构被修改的次数（The number of times this list has been structurally modified.）
         *           - 具体的分析见补充知识第一节
         */
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * 【Lin.C】：扩容的实际处理
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        
        // 【Lin.C】：1.5倍的扩容
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        
        // 【Lin.C】：扩容完需要确认是否达到最小大小要求（比如：不指定的时候默认就是10，也就是说初始化数组最小从10开始）
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        
        // 【Lin.C】：这个就是数组超过最大限制了（MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8）
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
            
        // 【Lin.C】：最后就是底层数据的拷贝了
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    /**
     * 【Lin.C】：Arrays.copyOf的实现参考（先声明一个泛型数组，然后调用native方法实现拷贝）
     */
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

# 删

## remove
```java
/**
 * 【Lin.C】：移除一个元素
 */
public E remove(int index) {
    // 【Lin.C】：跟get一样，所有操作的第一步是检查index是否合法
    rangeCheck(index);

    // 【Lin.C】：还是那个后面说了很多次的一致性检查的标记
    modCount++;
    E oldValue = elementData(index);

    /**
     * 【Lin.C】：流程是这样的
     *   - 先判断删除参数是不是合法
     *   - 把原数组从index+1（也就是需删除节点后的所有元素）复制到原数组的index位置
     *   - 此时，需删除的元素已经被后面的覆盖了（相当于已经删除了）
     *   - 但是，还有个问题；因为前面的复制，导致最后一个元素现在有两份；于是就理所当然的对最后一个元素置空
     *   - 流程结束；把刚刚被删除的元素返回
     */
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 【Lin.C】：最终发现并不是真的remove，还是通过native的数组拷贝来实现的
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

## clear
```java
/**
 * 【Lin.C】：- 一个很明显的特征（modCount++）
 *            - clear就是清空的过程，清空本身不是把数组完成置空，而是逐个把里面的元素置空（可能是出于性能考虑）
 */
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```

# 改

## set
```java
/**
 * 【Lin.C】：之前很少用到的一个方法，实现并不复杂，还是有用处的
 */
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

# 查

## indexOf
```java
/**
 * 【Lin.C】：- 实际上contains方法也是通过调用这个方法来实现的（indexOf(o) >= 0;）
 *            - 需要注意的点：indexOf在对null和对普通对象判断时是不一样的逻辑（主要是==和equals的区别）
 */
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            // 【Lin.C】：null本身不分配内存，所以不存在堆内存的地址，没办法用equals
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            // 【Lin.C】：普通对象会在堆内存产生内存分配，所以可以直接比较地址
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

## forEach
```java
/**
 * 【Lin.C】：循环遍历的方法，用到了Consumer这个接口的特性，这里就不详细分析了
 */
@Override
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    @SuppressWarnings("unchecked")
    final E[] elementData = (E[]) this.elementData;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

## get
```java
/**
 * 【Lin.C】：get本身简单到不能再简单了，主要是要留意这么简单的方法还拆成两个子方法，所以时刻保持方法简洁是必须滴
 */
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```

# 其他方法

## sort
```java
/**
 * 【Lin.C】：排序也是经常用到的一个方法
 */
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}

/**
 * 【Lin.C】：最终是走到数组本身的排序方法中去了（后面研究算法的时候再细聊吧）
 */
public static <T> void sort(T[] a, int fromIndex, int toIndex,
                            Comparator<? super T> c) {
    if (c == null) {
        // 【Lin.C】：和else里面的逻辑类似
        sort(a, fromIndex, toIndex);
    } else {
        rangeCheck(a.length, fromIndex, toIndex);
        if (LegacyMergeSort.userRequested)
            // 【Lin.C】：据注释To be removed in a future release.
            legacyMergeSort(a, fromIndex, toIndex, c);
        else
            // 【Lin.C】：TimSort 是一个归并排序做了大量优化的版本；后期应该主要应用这个版本
            TimSort.sort(a, fromIndex, toIndex, c, null, 0, 0);
    }
}

public static void sort(Object[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
    if (LegacyMergeSort.userRequested)
        legacyMergeSort(a, fromIndex, toIndex);
    else
        ComparableTimSort.sort(a, fromIndex, toIndex, null, 0, 0);
}
```

## toArray
```java
/**
 * 【Lin.C】：数组拷贝+泛型实现的，内部逻辑并不复杂，不啰嗦了
 */
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

# 内部类

## Itr
```java
/**
 * An optimized version of AbstractList.Itr
 * 【Lin.C】：多线程操作时应该会引发线程不安全性的问题
 */
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        // 【Lin.C】：检查状态一致性的操作（可能会抛异常）
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

    public void remove() {
        // 【Lin.C】：lastRet默认等于0，比如删除第一个元素，那么会先next，这样lastRet就等于0了
        if (lastRet < 0)
            throw new IllegalStateException();
        
        // 【Lin.C】：然后就是先检查两个modCount
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            
            // 【Lin.C】：操作完成后，把modCount两个值进行复位
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * 【Lin.C】：- 这个升级版的forEach性能比Iterator本身的性能是有所提升的
     *            - 这里用到了一个Consumer接口，这是一个函数式接口，后面新文章再单独对这个概念进行分析吧
     *            - 注意：forEach本身不是list接口提供的，而是Iterable提供的
     */
    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    /**
     * 【Lin.C】：一个神奇的一致性检查方法，在itr内部每个对集合结构改变的操作都会有这个检查
     */
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

## ListItr
```java
/**
 * An optimized version of AbstractList.ListItr
 * 【Lin.C】：这个类和前面的Itr基本是类似的存在，主要还是用来遍历List的，实际它也是继承Itr的，可以说是扩展版的Itr
 */
private class ListItr extends Itr implements ListIterator<E> {
    ListItr(int index) {
        super();
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor - 1;
    }

    @SuppressWarnings("unchecked")
    public E previous() {
        checkForComodification();
        int i = cursor - 1;
        if (i < 0)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i;
        return (E) elementData[lastRet = i];
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.set(lastRet, e);
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            ArrayList.this.add(i, e);
            cursor = i + 1;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}
```

# 关于modCount

## 基本说明
&emsp;&emsp;在前面一章的add方法中，我们第一次注意到一个来自于AbstractList的属性modCount，这个值呢是用来记录ArrayList结构变化次数的，也就是说add()、remove()、addAll()、removeRange()及clear()这些方法，每调用一次，modCount的值就加1。
&emsp;&emsp;那么这个值有什么作用呢？官话说是jdk在面对迭代遍历的时候为了避免不确定性而采取的快速失败原则。看上去说的简单明了，但对于这个概念还是很模糊。
&emsp;&emsp;我们知道，该字段被Iterator以及ListIterator的实现类所使用，如果该值被意外更改，Iterator或者ListIterator 将抛出ConcurrentModificationException异常；那么我们就结合ArrayList内的代码实现看看是怎么用的，想必就清楚了这个值是干什么的了。
```java
    /**
     * 【Lin.C】：重点是第一行，也就是通常通过迭代器操作时，第一行都会走这个checkForComodification
     */
    public E next() {
        checkForComodification();
        ......
    }
    
    /**
     * 【Lin.C】：这里的实现也很简要，就是判断当前的modCount和我期望的是不是一致，不一致就抛异常（够暴力）
     */
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
```

## ConcurrentModificationException
&emsp;&emsp;从上面的一节，我们已经发现当迭代集合时，某些不当的操作会引发ConcurrentModificationException，那具体哪几种操作会引发这个异常呢？
```java
/**
 * 【Lin.C】：出异常的第一种情况；通过iterator遍历时，不通过迭代器本身删除集合元素
 */
private static void operate1(List<String> list) {
    Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()) {
        String temp = iterator.next();
        if (iterator.next().equals("C")) {
            // 【Lin.C】：这个操作可以正确移除集合元素
            //iterator.remove();
            
            // 【Lin.C】：这个操作会抛出ConcurrentModificationException
            list.remove(temp);
        }
    }
}

/**
 * 【Lin.C】：出异常的第二种情况，在循环中直接remove元素
 */
private static void operate2(List<String> list) {
    List<String> list = new ArrayList<>();
    list.add("A");
    list.add("B");

    int i = 1;
    for (String v : list) {
        if ("A".equals(v)) {
            list.remove(v);
        }
    }
}

/**
 * 【Lin.C】：出异常的第三种情况，多线程操作同一个集合时
 */
private static void operate3(final List<String> list) {

    // 【Lin.C】：第一个线程，遍历集合，每次休眠10ms
    new Thread(new Runnable() {
        @Override
        public void run() {
            Iterator<String> iterator = list.iterator();
            while (iterator.hasNext()) {
                System.out.println(iterator.next());
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }).start();
    
    // 【Lin.C】：第二个线程，遍历同一个集合，然后做一个条件remove
    new Thread(new Runnable() {
        @Override
        public void run() {
            // 【Lin.C】：注意，这种遍历方式，本身是不会出现异常的
            for (int i = 0; i < list.size(); i++) {
                if (list.get(i).equals("A")) {
                    list.remove(i);
                }
            }
        }
    }).start();
}

/**
 * 【Lin.C】：不出异常的第一种情况，简单的for循环遍历（适用于增删集合元素）
 */
private static void operate4(List<String> list) {
    List<String> list = new ArrayList<>();
    list.add("A");
    list.add("B");

    for (int i = 1; i < list.size(); i++) {
        if (i == 1) {
            list.remove(list.get(i));
        }
    }
}

/**
 * 【Lin.C】：不出异常的第二种情况，迭代器操作
 */
private static void operate5(List<String> list) {
    Iterator<String> iterator = list.iterator();
    while (iterator.hasNext()) {
        if (iterator.next().equals("1")) {
            iterator.remove();
        }
    }
}
```

## 异常过程追踪
&emsp;&emsp;针对上一节提到的ConcurrentModificationException，我们已经总结了某些场景下会出现这个异常，但是为什么出现呢？通过一次debug我们来了解一下里面的玄机。
```java
/**
 * 测试Demo
 * @param args
 */
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("A");
    list.add("B");
    list.add("C");

    int i = 1;
    for (String v : list) {
        if (i++ == 1) {
            // 【Lin.C】：这个操作是会返回true的
            System.out.println(list.remove(v));
        }
    }
}
```

### remove的流程是怎么走的？
```
- list.remove(v);发起调用
- 走到ArrayList的public boolean remove(Object o)方法
- 然后remove方法内部调用了fastRemove(index);
- fastRemove内部首先是modCount++;然后就是数组拷贝操作
- 至此remove流程完成（其实没有涉及到checkForComodification）
```

### 异常是如何引发的呢？
```
- 按照上面的删除逻辑，实际上程序走到remove完成这一步是不会报异常的；而且删除操作本身也是成功的
- 真正引发异常的是删除完之后的for循环取下一个元素导致的，因为这种类foreach的for循环的过程调用了itr的next
- next的第一行是checkForComodification();此时因为remove对modCount加了1，而expectedModCount还是未加1之前的值
- 所以if (modCount != expectedModCount)是true，也就是验证出现数据不一致性，抛出了这个异常
```

### 普通for和迭代器为什么不会引发异常？
```
- 普通for循环：因为这种for直接通过下标索引，不会走到itr内部的next，也就不会触发checkForComodification
- 迭代器remove：因为itr内部的remove方法在最后都会expectedModCount = modCount;这就保证了modCount值remove后的一致性
```

# 扩展：Vector
&emsp;&emsp;前面我们就发现了，ArrayList本身是线程不安全的，所以有了Vector；Vector就是线程安全版本的数组容器。Vector的实现很简单，就是把所有的方法统统加上synchronized，通过下面的方法对比图也很明显可以看得出来这个特征。
&emsp;&emsp;也可以不使用Vector，用Collections.synchronizedList把一个普通ArrayList包装成一个线程安全版本的数组容器也可以，原理同Vector是一样的，就是给所有的方法套上一层synchronized。（关于Collections后面会再单独列一篇文章解读）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-08-11-02.jpg" width="90%">