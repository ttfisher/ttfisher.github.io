---
title: 畅读源码系列之JDK（四）：HashMap
comments: true
categories:
  - 【201】蓦然回首灯火阑珊处之JDK
tags:
  - 畅读源码
abbrlink: 94cbc8cc
date: 2018-08-12 16:34:00
---
【引言】针对线性结构的两类主要集合前面已经做了解读，这一章里面，我们来阅读阅读另外一种key-value模式的数据结构，日常编码中我们最常用的想必就是HashMap了！
<div align=center><img src="/img/2018/2018-08-12-03.jpg" width="500"/></div>
<!-- more -->

# 类的定义
```java
/**
 * 【Lin.C】： - 溯源的话，还是回到Map；继承关系：HashMap->AbstractMap->Map
 *             - 其他一些可clone、可序列化的特性不提也罢
 * @since   1.2
 */
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable 
{
}
```
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-22-06.jpg" width="60%">

# 成员变量
```java
    // 【Lin.C】：字面意思就可以理解：默认容量
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    // 【Lin.C】：最大容量，这个地方有个重点MUST be a power of two（参考后面的补充说明）
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 【Lin.C】：负载因子，计算扩容临界值的默认负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    // 【Lin.C】：由链表转换成树的阈值
    static final int TREEIFY_THRESHOLD = 8;
 
    // 【Lin.C】：由树转换成链表的阈值
    static final int UNTREEIFY_THRESHOLD = 6;

    // 【Lin.C】：当桶中的bin被树化时最小的hash表容量；至少是TREEIFY_THRESHOLD的4倍。
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    // 【Lin.C】：所谓的数组+链表/红黑树里面的数组结构（扩容也是针对这个table的）
    transient Node<K,V>[] table;
    
    // 【Lin.C】：The next size value at which to resize (capacity * load factor).
    int threshold;

    // 【Lin.C】：entry集合
    transient Set<Map.Entry<K,V>> entrySet;

    // 【Lin.C】：Map的大小（有多少元素了）
    transient int size;

    // 【Lin.C】：就是那个什么每次结构变更都会更新的计数器
    transient int modCount;
```

# 构造方法

## HashMap()
```java
    // 【Lin.C】：平时我们最常用的一种构造方法，都使用默认参数
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```

## HashMap(int initialCapacity)
```java
    /**
     * 【Lin.C】：指定初始化大小，实际调用的还是下一个构造方法
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

## HashMap(int initialCapacity, float loadFactor)
```java
    /**
     * 【Lin.C】：比较核心的一个构造方法，指定初始化大小，指定负载因子（实际这里也还没有初始化table）
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
        this.loadFactor = loadFactor;
        
        // tableSizeFor是用来计算大于initialCapacity的最小的2的N次幂的值
        this.threshold = tableSizeFor(initialCapacity);
    }
    
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

## HashMap(Map<? extends K, ? extends V> m)
```java
    /**
     * 【Lin.C】：从其他Map来初始化，也就是把元素都put过来
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

# 增

## put
```java
    /**
     * 【Lin.C】：put本身很简单，传入的是key和value两个参数，返回值若是key已经存在map里面了，返回当前已存在的value，否则返回null
     */
    public V put(K key, V value) {
        // 【Lin.C】：这里算的是key的hash
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * 【Lin.C】：这里做了个16位右移异或操作
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    
    /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
                   
        // 【Lin.C】：一些变量声明（tab就对应当前的数组，p用于临时存储当前table中碰撞hash所得到的位置的Node）
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        
        // 【Lin.C】：如果当前map中无数据，执行resize方法，并且返回n，n就是resize后的大小
        if ((tab = table) == null || (n = tab.length) == 0)
            // 【Lin.C】：这里做了3个操作（resize，把结果赋值给tab，把tab的长度赋值给n
            n = (tab = resize()).length;
            
        // 【Lin.C】：判断本hash应该落在table的哪个index下；如果要插入的Node存放的这个位置刚好没有元素（=null），进入内部逻辑
        if ((p = tab[i = (n - 1) & hash]) == null)
        
            // 【Lin.C】：那么把这个键值对封装成Node对象，直接作为这个位置的链表头了（第四个参数null就是Node的next属性了）
            tab[i] = newNode(hash, key, value, null);
        
        // 【Lin.C】：走这个分支的话，就表示即将落的数组上已经有别的元素了
        else {
            
            Node<K,V> e; K k;
            
            // 【Lin.C】：数组中当前位置正好和要存的key是相同的（hash相同，key相同），那么就把当前Node记录下来到e里面
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            
            // 【Lin.C】：如果p是一个红黑树，那么走入putTreeVal的流程（后面说）
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            
            // 【Lin.C】：走到这边的话，就是没有在链表头命中重复key的而且也没有达到红黑树大小的结构
            else {
            
                // 【Lin.C】：做个循环遍历当前这条链表上的所有数据
                for (int binCount = 0; ; ++binCount) {
                    
                    // 【Lin.C】：如果p的下一个元素为空了
                    if ((e = p.next) == null) {
                        
                        // 【Lin.C】：那么很自然的把当前元素接上去
                        p.next = newNode(hash, key, value, null);
                        
                        // 【Lin.C】：这里就要判断要不要转成树了（因为binCount是从0开始的，所以后面有个-1）
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            
                            // 【Lin.C】：数据结构转换
                            treeifyBin(tab, hash);
                        
                        // 【Lin.C】：哦了，都已经把元素塞进去了，就跳出循环吧
                        break;
                    }
                    // 【Lin.C】：e != null才会继续往下走
                    
                    // 【Lin.C】：如果当前这个Node正好和需要存储的元素同key（hash相同，key相同），那么就也跳出循环（重复了嘛）
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    
                    // 【Lin.C】：因为第一步e = p.next；实际就已经往后移了一个Node了，这里再把它赋值给临时变量p到下一轮比较
                    p = e;
                }
            }
            
            // 【Lin.C】：这里e不为空实际表示链表里面肯定有key命中了这个hash（因为上面的for循环只有两种结束条件：e为空、e命中了这个hash）
            if (e != null) { // existing mapping for key
            
                V oldValue = e.value;
                
                // 【Lin.C】：这里的逻辑就是处理相同key的时候保留已有值还是新来的值的问题了；还有个特殊场景就是oldValue==null是肯定会更新的
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                
                // 【Lin.C】：这个方法啥在hashmap里面也没做，但是它在LinkedHashMap类里面是有实现的（包括afterNodeInsertion、afterNodeRemoval）
                afterNodeAccess(e);
                
                // 【Lin.C】：返回老的原始的Value
                return oldValue;
            }
        }
        
        // 【Lin.C】：结构变化计数器
        ++modCount;
        
        // 【Lin.C】：加了一个元素size肯定要对应+1了，加完还得看看要不要扩容（扩容就会重新计算threshold）
        if (++size > threshold)
            
            // 【Lin.C】：这个就是扩容方法本尊了，后面会单独剖析一下
            resize();
        
        // 【Lin.C】：跟afterNodeAccess一样的目前啥也不做的空方法
        afterNodeInsertion(evict);
        
        // 【Lin.C】：返回空（对应(e = p.next) == null这个逻辑就会返回空，也就是这个key是新的）
        return null;
    }
```

# 删

## remove
```java
    /**
     * 【Lin.C】：都是以key的hash为起点
     */
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
    }
    
    /**
     * Implements Map.remove and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to match if matchValue, else ignored
     * @param matchValue if true only remove if value is equal
     * @param movable if false do not move other nodes while removing
     * @return the node, or null if none
     */
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
                               
        // 【Lin.C】：一些变量声明（tab就对应当前的数组，p用于临时存储当前table中碰撞hash所得到的位置的Node）
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        
        // 【Lin.C】：table中当前hash对应的Node必须不为空才可以remove
        if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
        
            Node<K,V> node = null, e; K k; V v;
            
            // 【Lin.C】：如果第一个元素就命中了key了，就记下来
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            
            // 【Lin.C】：下一个元素不为空的情况
            else if ((e = p.next) != null) {
            
                // 【Lin.C】：如果是树，走到树的get方法获取Node
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                    
                // 【Lin.C】：不是树那就是链表了
                else {
                    
                    // 【Lin.C】：循环往后找
                    do {
                        
                        // 【Lin.C】：某个元素和当前要remove的元素匹配了（hash相同，key相同），则找到了，break出循环
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        
                        // 【Lin.C】：e控制链表挨个往下走，并把每走一步最新的Node赋给了p（后面remove有用到）
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            
            // 【Lin.C】：有命中元素（matchValue控制待删除元素的value和参数传进来的value是不是相等才删除）
            if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {
                
                // 【Lin.C】：树走树的remove逻辑
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                
                // 【Lin.C】：在第一个元素就命中了才会node==p（待删除元素在桶链表的表头）
                else if (node == p)
                    // 【Lin.C】：把node的下一个元素放到table中（也就是node被移除了）
                    tab[index] = node.next;
                    
                // 【Lin.C】：do-while逻辑命中的走到这里了（待删除元素在桶链表的中间）
                else
                    // 【Lin.C】：node就是找到的key匹配的元素，p实际是node的前一个元素，这里就实现了把node断链的操作
                    p.next = node.next;
                    
                // 【Lin.C】：结构变化计数器
                ++modCount;
                
                // 【Lin.C】：size变化
                --size;
                
                // 【Lin.C】：空方法
                afterNodeRemoval(node);
                
                // 【Lin.C】：返回被remove的元素
                return node;
            }
        }
        
        // 【Lin.C】：不存在这个元素，没法remove，返回null
        return null;
    }
```

## clear
```java
    /**
     * 【Lin.C】：Removes all of the mappings from this map.
     */
    public void clear() {
        Node<K,V>[] tab;
        modCount++;
        
        // 【Lin.C】：把table的所有元素清空，size归零；即完成了clear操作
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i)
                tab[i] = null;
        }
    }
```

# 改

## replace
```java
    /**
     * 【Lin.C】：There is absolutely no difference in put and replace when there is a current mapping for the wanted key.
     */
    @Override
    public V replace(K key, V value) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) != null) {
            V oldValue = e.value;
            e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
        return null;
    }
    
    @Override
    public boolean replace(K key, V oldValue, V newValue) {
        Node<K,V> e; V v;
        if ((e = getNode(hash(key), key)) != null &&
            ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
            e.value = newValue;
            afterNodeAccess(e);
            return true;
        }
        return false;
    }
```

# 查

## get
```java
    /**
     * 【Lin.C】：都是以key的hash为起点
     */
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    
    /**
     * Implements Map.get and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @return the node, or null if none
     */
    final Node<K,V> getNode(int hash, Object key) {
        // 【Lin.C】：一些变量声明
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        
        // 【Lin.C】：table中当前hash对应的Node必须不为空才可以get
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
            
            // 【Lin.C】：每次都先检查一下第一个元素，如果命中了，就返回了
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            
            // 【Lin.C】：第一个元素没有命中，就开始遍历链表/树
            if ((e = first.next) != null) {
                
                // 【Lin.C】：树走树的get逻辑
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);

                // 【Lin.C】：循环链表直至队尾
                do {
                    // 【Lin.C】：命中了（hash相等，key相等）就返回
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        
        // 【Lin.C】：前面都没get到那就是不存在这个Node了
        return null;
    }
```

## forEach
```java
    /**
     * 【Lin.C】：挺好用，但是这个Consumer暂时还没怎么研究过，后面需要研究研究
     */
    @Override
    public void forEach(BiConsumer<? super K, ? super V> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.key, e.value);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
```

## isEmpty
```java
    /**
     * 【Lin.C】：简单到注释都多余了（ArrayList和LinkedList也是一样的实现）
     */
    public boolean isEmpty() {
        return size == 0;
    }
```

## containsKey
```java
    /**
     * 【Lin.C】：实际还是回到了getNode方法，能取到元素就为true
     */
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```

## containsValue
```java
    /**
     * 【Lin.C】：真没怎么用过这个方法（数据多的话这个效率应该偏低）
     */
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
        
            // 【Lin.C】：对table的每个元素进行循环
            for (int i = 0; i < tab.length; ++i) {
                
                // 【Lin.C】：对元素内部的链表进行循环，然后比较value是不是相等
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value || (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```

# 扩容

## resize
```java
    /**
     * 【Lin.C】：核心方法（扩容）-- 除了初始化特殊，后面每次都Double
     */
    final Node<K,V>[] resize() {
        
        // 【Lin.C】：当前table（老）对象
        Node<K,V>[] oldTab = table;
        
        // 【Lin.C】：老table的容量
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        
        // 【Lin.C】：即将扩容的容量大小
        int oldThr = threshold;
        
        // 【Lin.C】：一些变量声明
        int newCap, newThr = 0;
        
        // 【Lin.C】：老table不为空（map里面有元素了）
        if (oldCap > 0) {
        
            // 【Lin.C】：判断是否越界了
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            
            // 【Lin.C】：如果oldCap大于最小默认容量，且扩容一倍后仍然合法
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                
                // 【Lin.C】：新数组长度扩大为原来的2倍
                // 【Lin.C】：临界值也扩大为原来的2倍
                newThr = oldThr << 1; // double threshold
        }
        
        // 【Lin.C】：老table为空，但扩容大小限制大于0（第一次带参数初始化时）
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        
        // 【Lin.C】：老table为空，而且也没有指定扩容大小（第一次不带参数初始化时）
        else {               // zero initial threshold signifies using defaults
            
            // 【Lin.C】：默认容量16
            newCap = DEFAULT_INITIAL_CAPACITY;
            
            // 【Lin.C】：默认扩容大小：0.75 * 16 = 12
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        
        // 【Lin.C】：如果扩容大小限制还是为0（也就是没有初始化参数）
        if (newThr == 0) {
            
            // 【Lin.C】：当前大小 * 负载因子
            float ft = (float)newCap * loadFactor;
            
            // 【Lin.C】：上限判断
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
        }
        
        // 【Lin.C】：下一轮扩容临界值敲定（比如：putVal里面resize前的判断++size > threshold）
        threshold = newThr;
        
        /**
         * 【Lin.C】：如果是最开始还没有元素（初始化）：
         *              - 如果初始化的时候带了参数；那么newCap=initialCapacity，threshold=(int)(initialCapacity*loadFactor)
         *              - 如果初始化的时候没带参数；那么initialCapacity = 16，threshold = 12（默认的）
         *            如果Map里面已经有元素了：那么直接扩容2倍（threshold也扩大两倍）
         */
        
        // 【Lin.C】：新创建一个Node数组，并赋值给table
        @SuppressWarnings({ "rawtypes", "unchecked" })
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        
        // 【Lin.C】：接下来理论上猜测应该就是移动数据了
        if (oldTab != null) {
        
            // 【Lin.C】：逐个遍历老table
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                
                // 【Lin.C】：遍历的元素不为空则进入处理逻辑
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    
                    // 【Lin.C】：如果这个位置就一个元素，那么直接扔到新table里去
                    if (e.next == null)
                        
                        // 【Lin.C】：还是通过&操作确定存放在哪个位置（数组的index）
                        newTab[e.hash & (newCap - 1)] = e;
                    
                    // 【Lin.C】：如果该节点是树，就走树的逻辑处理
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    
                    // 【Lin.C】：进入这个逻辑的就是不止一个元素的链表了
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        
                        // 【Lin.C】：循环那是必不可少的
                        do {
                            
                            /**
                             * 【Lin.C】：此段注释来源于CSDN某博主
                             * 注意：不是(e.hash & (oldCap-1));而是(e.hash & oldCap)
                             * 
                             * (e.hash & oldCap) 得到的是元素在数组中的位置是否需要移动的标记，示例如下
                             *
                             * 示例1：
                             * e.hash=10 0000 1010
                             * oldCap=16 0001 0000
                             *   &   =0  0000 0000       比较高位的第一位 0
                             * 结论：元素位置在扩容后数组中的位置没有发生改变
                             *
                             * 示例2：
                             * e.hash=17 0001 0001
                             * oldCap=16 0001 0000
                             *   &   =1  0001 0000      比较高位的第一位   1
                             * 结论：元素位置在扩容后数组中的位置发生了改变，新的下标位置是原下标位置+原数组长度
                             *
                             * 老下标位置
                             *   e.hash=10 0000 1010
                             * oldCap-1=15 0000 1111
                             *      &  =10 0000 1010
                             *   e.hash=17 0001 0001
                             * oldCap-1=15 0000 1111
                             *      &  =1  0000 0001
                             * 
                             * 新下标位置
                             *   e.hash=17 0001 0001
                             * newCap-1=31 0001 1111    newCap=32
                             *      &  =17 0001 0001    1+oldCap = 1+16
                             *
                             * 元素重新计算hash后，因为n变为2倍，那么n-1的mask范围在高位多1bit，因此新的index就会发生这样的变化：
                             * 0000 0001 -> 0001 0001
                             */
                            next = e.next;
                            
                            // 【Lin.C】：元素位置不用移动
                            if ((e.hash & oldCap) == 0) {
                                
                                // 【Lin.C】：循环第一次会进入此逻辑，取到head
                                if (loTail == null)
                                    loHead = e;
                                
                                // 【Lin.C】：后续循环就不断地往tail后面挂元素（从第二个循环开始就会走else）
                                else
                                    loTail.next = e;
                                    
                                /**
                                 * 【Lin.C】：每次链接完之后，把tail本身赋值为当前值
                                 *   - 在第二次之后，loTail指向了和loTail.next(loHead.next.next)相同的内存（可以理解为自引用了一遍）
                                 */
                                loTail = e;
                            }
                            
                            // 【Lin.C】：元素位置需要移动（遍历逻辑都一样）
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        
                        // 【Lin.C】：不用移动位置的链表（直接放在原index下即可）
                        if (loTail != null) {
                        
                            // 【Lin.C】：尾节点置空（为了剔除上面那个自引用）
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        
                        // 【Lin.C】：需要移动位置的链表（移到新table的j + oldCap位置）
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                        
                        // 【Lin.C】：其实链表相对位置本身没有动，只是把链表挂到一个新的table里面，也就是只变了一下内存地址而已
                    }
                }
            }
        }
        
        // 【Lin.C】：大功告成，return了
        return newTab;
    }
```

# 内部类

## Node
```java
    /**
     * 【Lin.C】：链表的节点；首先Node是Map存储的基础节点，其次它是Entry的实现类；代码很简单，就不细注释了
     */
    static class Node<K,V> implements Map.Entry<K,V> {

        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

## TreeNode
```java
    /**
     * 【Lin.C】：红黑树的节点
     */
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }

        /**
         * Returns root of tree containing this node.
         */
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }
        
        ......
    }
```

## EntrySet
```java
    /**
     * 【Lin.C】：常用的遍历方法entrySet
     */
    public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
    }

    /**
     * 【Lin.C】：EntrySet的结构（封装了Iterator和forEach）
     */
    final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<Map.Entry<K,V>> iterator() {
            return new EntryIterator();
        }
        public final boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Node<K,V> candidate = getNode(hash(key), key);
            return candidate != null && candidate.equals(e);
        }
        public final boolean remove(Object o) {
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>) o;
                Object key = e.getKey();
                Object value = e.getValue();
                return removeNode(hash(key), key, value, true, true) != null;
            }
            return false;
        }
        public final Spliterator<Map.Entry<K,V>> spliterator() {
            return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

## KeySet
```java
    /**
     * 【Lin.C】：类似于EntrySet，只不过这个Set只有Key
     */
    final class KeySet extends AbstractSet<K> {
        public final int size()                 { return size; }
        public final void clear()               { HashMap.this.clear(); }
        public final Iterator<K> iterator()     { return new KeyIterator(); }
        public final boolean contains(Object o) { return containsKey(o); }
        public final boolean remove(Object key) {
            return removeNode(hash(key), key, null, false, true) != null;
        }
        public final Spliterator<K> spliterator() {
            return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
        }
        public final void forEach(Consumer<? super K> action) {
            Node<K,V>[] tab;
            if (action == null)
                throw new NullPointerException();
            if (size > 0 && (tab = table) != null) {
                int mc = modCount;
                for (int i = 0; i < tab.length; ++i) {
                    for (Node<K,V> e = tab[i]; e != null; e = e.next)
                        action.accept(e.key);
                }
                if (modCount != mc)
                    throw new ConcurrentModificationException();
            }
        }
    }
```

# MUST be a power of two约定
&emsp;&emsp;关于MUST be a power of two这个话题，我觉得有必要谈一谈；
```java
// JDK1.8之前呢，是这么算数组下标的（h就是hash）
static int indexFor(int h, int length) {
    return h & (length-1);
}

// 但是在1.8里面indexFor这个方法就找不到了，后来发现是直接写到了方法里面了，就是下面的(n - 1) & hash
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
if ((p = tab[i = (n - 1) & hash]) == null)
```
&emsp;&emsp;HashMap的初始容量和扩容都是以2的次方来进行的，那么length-1换算成二进制的话肯定所有位都为1，就比如2的3次方为8，length-1的二进制表示就是111， 而按位与计算的原则是两位同时为“1”，结果才为“1”，否则为“0”。所以h & (length-1)运算从数值上来讲其实等价于对length取模，也就是h%length。
&emsp;&emsp;那我们反证一下，如果不满足前提条件“HashMap的初始容量和扩容都是以2的次方来进行的”，会发生什么问题呢？
&emsp;&emsp;假设当前table的length是15，二进制表示为1111，那么length-1就是1110，此时有两个hash值为8和9的key需要计算索引值，计算过程如下：
```
8的二进制表示：1000
8&（length-1）= 1000 & 1110 = 1000，索引值即为8;

9的二进制表示：1001
9&（length-1）= 1001 & 1110 = 1000，索引值也为8;
```
&emsp;&emsp;这样一来就产生了相同的索引值，也就是说两个hash值为8和9的key会定位到数组中的同一个位置上形成链表，这就产生了碰撞；而查询的时候需要遍历这个链表，这样就降低了查询的效率。
&emsp;&emsp;同时，我们也可以发现，当数组长度为15的时候，hash值会与length-1（1110）进行按位与，那么最后一位永远是0，而0001，0011，0101，1001，1011，0111，1101这几个位置永远都不能存放元素了，会造成严重的空间浪费，更糟的是这种情况下，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率。
&emsp;&emsp;因此可以看出，只有当数组长度为2的n次方时，不同的key计算得出的index索引相同的几率才会较小，数据在数组上分布也比较均匀，碰撞的几率也小，相对的，查询的时候就不用遍历某个位置上的链表，这样查询效率也就较高了。
&emsp;&emsp;此外，位运算快于十进制运算，hashmap扩容也是按位扩容，这样同时也提高了运算效率。
&emsp;&emsp;所以，还是一切为了效率，尽可能的达到最优。

# 关于put的返回值验证
```java
package com.example.demo;

import java.util.HashMap;

/**
 * Created by chenglin on 2018/8/23.
 */
public class Test {

    public static void main(String[] args) {
        HashMap hm = new HashMap<>();
        System.out.println(hm.put("A", "a"));
        System.out.println(hm.put("B", "b"));
        System.out.println(hm.put("A", "aa"));
    }
}

[Console log]:
null
null
a
```

# 关于Hashtable
&emsp;&emsp;Hashtable本身现在已经基本处于被抛弃的阶段，但是了解一下之后发现也是比较暴力的解决了同步的问题，就是在方法上直接加synchronized。
```java
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```