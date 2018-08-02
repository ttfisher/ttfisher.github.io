---
title: 说说集合系列（二）：Collection
comments: true
categories:
  - JDK one by one - collection plus
tags:
  - 集合
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


## List的不同遍历方式
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

# Set

# Queue