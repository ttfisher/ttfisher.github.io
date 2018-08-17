---
title: 畅读源码系列之JDK（四）：HashMap（未完成...)
comments: true
categories:
  - Source Code Reading - JDK
tags:
  - 畅读源码
abbrlink: 94cbc8cc
date: 2018-08-12 16:34:00
---
【引言】针对线性结构的两类主要集合前面已经做了解读，这一章里面，我们来阅读阅读另外一种key-value模式的数据结构，日常编码中我们最常用的想必就是HashMap了！
<div align=center><img src="/img/2018-08-12-03.jpg" width="500"/></div>
<!-- more -->

# 类基本定义

## HashMap
```java
/**
 * 【CHENG】： - 溯源的话，还是回到Map；继承关系：HashMap->AbstractMap->Map
 *             - 其他一些可clone、可序列化的特性不提也罢
 * @since   1.2
 */
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable 
{
}
```

# 属性和构造方法

## 基本属性
```java
    /**
     * 【CHENG】：字面意思就可以理解：默认容量
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * The maximum capacity, used if a higher value is implicitly specified
     * by either of the constructors with arguments.
     * MUST be a power of two <= 1<<30.
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The load factor used when none specified in constructor.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

## 构造方法一
```java
    /**
     * 【CHENG】：平时我们最常用的一种构造方法，不指定大小；默认初始化一个空数组，第一次add就会引发扩容
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

# 其他常用方法

## add
```java
    /**
     * 【CHENG】：流程很简单
     */
```