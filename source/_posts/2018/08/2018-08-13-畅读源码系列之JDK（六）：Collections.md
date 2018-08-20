---
title: 畅读源码系列之JDK（六）：Collections（未完成...)
comments: true
categories:
  - Source Code Reading - JDK
tags:
  - 畅读源码
abbrlink: 1ebf7315
date: 2018-08-13 10:34:00
---
【引言】前一篇说完了ArrayList，这一篇就来说说和ArrayList形影不离，形同姊妹的另外一种结构：LinkedList！
<div align=center><img src="/img/2018/2018-08-13-01.jpg" width="500"/></div>
<!-- more -->

# 类基本定义

## ArrayList
```java
/**
 * 【CHENG】：从ArrayList本身的定义出发，可以简单分析出它的基本特性
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

# 属性和构造方法

## 基本属性
```java
    /**
     * 【CHENG】：字面意思就可以理解：默认容量
     */
    private static final int DEFAULT_CAPACITY = 10;
    
    /**
     * 【CHENG】：默认的空实例
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    /**
     * 【CHENG】：实际存储的数组（因为ArrayList底层就是数组结构）
     */
    transient Object[] elementData; // non-private to simplify nested class access
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