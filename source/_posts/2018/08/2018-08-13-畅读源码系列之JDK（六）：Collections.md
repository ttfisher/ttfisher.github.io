---
title: 畅读源码系列之JDK（六）：Collections
comments: true
categories:
  - 技术结构升级
  - JDK源码汇
tags:
  - 畅读源码
abbrlink: 1ebf7315
date: 2018-08-13 10:34:00
---
【引言】Collections和Collection说实话不注意的话有时候真的会混淆，其实二者差别还是很大的，Collections作为一个集合辅助工具而存在的，其实里面提供了很多适合我们的工具，说到工具其实apache的很多工具包也很有用的，减少了我们很多重复工作，真的有必要探究探究！
<div align=center><img src="https://github.com/ttfisher/images/raw/master/2018/2018-08-13-01.jpg" width="55%"/></div>
<!-- more -->

# 类的定义

## Collections
```java
/**
 * 【Lin.C】：简单明了，连构造方法也很简单明了
 */
public class Collections {
    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {
    }
}
```

# 常用方法

## sort
```java
    /**
     * 【Lin.C】：sort是个使用频率比较高的方法了，用来排序（但前提是你的list要实现Comparable接口）
     */
    public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }
    
    public interface Comparable<T> {
        /**
         * Compares this object with the specified object for order.  Returns a
         * negative integer, zero, or a positive integer as this object is less
         * than, equal to, or greater than the specified object.
         */
        public int compareTo(T o);
    }
```

## add
```java
    /**
     * 【Lin.C】：为一个集合添加元素（试了下比挨个add好用的多了）
     */
    public static <T> boolean addAll(Collection<? super T> c, T... elements) {
        boolean result = false;
        for (T element : elements)
            result |= c.add(element);
        return result;
    }

[Demo]：
    public class Test {

        public static void main(String[] args) {
            List<String> l1 = new ArrayList<>();
            l1.add("l1-a");
            l1.add("l1-b");

            Collections.addAll(l1, "c", "d");
            System.out.println(l1);
        }
    }

[Console log]：
[l1-a, l1-b, c, d]
```

## binarySearch
```java
    /**
     * 【Lin.C】：二分法查找
     */
    public static <T> int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }

[Demo]：
    public class Test {

        public static void main(String[] args) {
            List<String> list = new ArrayList<>();
            Collections.addAll(list, "a", "b", "c", "d");
            System.out.println(Collections.binarySearch(list, "c"));
        }
    }

[Console log]：
2
```

## copy
```java
    /**
     * 【Lin.C】：复制一个list
     */
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }
```

## emptyList
```java
    /**
     * 【Lin.C】：置空一个list（EMPTY_LIST也是一个内部类）
     */
    public static final <T> List<T> emptyList() {
        return (List<T>) EMPTY_LIST;
    }
```

## reverse
```java
    /**
     * 【Lin.C】：反转集合，有时候还是有些用处的
     */
    public static void reverse(List<?> list) {
        int size = list.size();
        if (size < REVERSE_THRESHOLD || list instanceof RandomAccess) {
            for (int i=0, mid=size>>1, j=size-1; i<mid; i++, j--)
                swap(list, i, j);
        } else {
            // instead of using a raw type here, it's possible to capture
            // the wildcard but it will require a call to a supplementary
            // private method
            ListIterator fwd = list.listIterator();
            ListIterator rev = list.listIterator(size);
            for (int i=0, mid=list.size()>>1; i<mid; i++) {
                Object tmp = fwd.next();
                fwd.set(rev.previous());
                rev.set(tmp);
            }
        }
    }
```

## synchronizedXXX
```java
    //【Lin.C】：synchronized一个系列有很多个方法，比如以SynchronizedCollection为例
    public static <T> Collection<T> synchronizedCollection(Collection<T> c) {
        
        //【Lin.C】：实现原理基本类似（都是通过一个内部类转换的）
        return new SynchronizedCollection<>(c);
    }
    
    //【Lin.C】：实际就是在所有方法执行的时候加了个synchronized (mutex)锁而已
    //【Lin.C】：其他比如SynchronizedMap等也都是依样画葫芦来实现的
    static class SynchronizedCollection<E> implements Collection<E>, Serializable {
        private static final long serialVersionUID = 3053995032091335093L;

        final Collection<E> c;  // Backing Collection
        final Object mutex;     // Object on which to synchronize

        SynchronizedCollection(Collection<E> c) {
            this.c = Objects.requireNonNull(c);
            mutex = this;
        }

        SynchronizedCollection(Collection<E> c, Object mutex) {
            this.c = Objects.requireNonNull(c);
            this.mutex = Objects.requireNonNull(mutex);
        }

        public int size() {
            synchronized (mutex) {return c.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return c.isEmpty();}
        }
        public boolean contains(Object o) {
            synchronized (mutex) {return c.contains(o);}
        }
        ......
    }
```
