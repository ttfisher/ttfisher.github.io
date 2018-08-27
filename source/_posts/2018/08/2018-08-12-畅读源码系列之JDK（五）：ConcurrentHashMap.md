---
title: 畅读源码系列之JDK（五）：ConcurrentHashMap（未完成...)
comments: true
categories:
  - Source Code Reading - JDK
tags:
  - 畅读源码
abbrlink: 98444c68
date: 2018-08-12 17:22:00
---
【引言】ConcurrentHashMap，从名字上来看呢，是Concurrent和HashMap的结合体，实际就是线程安全的HashMap了，但是它如何实现这个线程安全呢？和Hashtable又有何区别呢？这里面就有的说道说道了。
<div align=center><img src="/img/2018/2018-08-12-04.jpg" width="500"/></div>
<!-- more -->

# 类的定义
```java
/**
 * 【CHENG】： - 继承关系1：ConcurrentHashMap->AbstractMap->Map
 *             - 继承关系2：ConcurrentHashMap->ConcurrentMap->Map
 * @since 1.5
 * @author Doug Lea
 */
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
{
}
```

# 重要常量
```java
    /* ---------------- Constants -------------- */

    // 【CHENG】：最大容量
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    // 【CHENG】：默认容量
    private static final int DEFAULT_CAPACITY = 16;

    // 【CHENG】：数组最大大小
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    // 【CHENG】：Unused but defined for compatibility with previous versions of this class.
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

    // 【CHENG】：负载因子
    private static final float LOAD_FACTOR = 0.75f;

    // 【CHENG】：链表转树的临界值
    static final int TREEIFY_THRESHOLD = 8;

    // 【CHENG】：树转链表的临界值
    static final int UNTREEIFY_THRESHOLD = 6;

    // 【CHENG】：当桶中的bin被树化时最小的hash表容量；至少是TREEIFY_THRESHOLD的4倍。
    static final int MIN_TREEIFY_CAPACITY = 64;

    // 【CHENG】：数据写入或移动时需要使用的常量
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    
    // 【CHENG】：在 spread() 方法中用来对 hashcode 进行高位hash减少可能发生的碰撞。
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
    
    // 【CHENG】：还有很多常量，跟此次解读关系不大的就不标注了
    ......
```

# 成员变量
```java
    /**
     * The array of bins. Lazily initialized upon first insertion.
     * Size is always a power of two. Accessed directly by iterators.
     */
    // 【CHENG】：第一层存储（数组），第一次insert时才会初始化，必须是power of two
    transient volatile Node<K,V>[] table;

    /**
     * The next table to use; non-null only while resizing.
     */
    // 【CHENG】：应该是在扩容是使用的
    private transient volatile Node<K,V>[] nextTable;

    /**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
    /**
     *【CHENG】：控制初始化和扩容的一个控制标记位；（既代表 HashMap 的 threshold；又代表进行扩容时的线程数）
     *           负数代表正在进行初始化或扩容操作 
     *               ① -1代表正在初始化 
     *               ② -N 表示有N-1个线程正在进行扩容操作 
     *           正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小 
    */
    private transient volatile int sizeCtl;

    /**
     * The next table index (plus one) to split while resizing.
     */
    // 【CHENG】：下一个transfer任务的起始下标index 加上1 之后的值
    private transient volatile int transferIndex;

    // 【CHENG】：以下三个变量需要结合java.util.concurrent.atomic.LongAdder理解
    // 【CHENG】：计数器基本值，主要在碰到多线程竞争时使用，需要通过CAS进行更新
    private transient volatile long baseCount;
    
    /**
     * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
     */
    // 【CHENG】：CAS自旋锁标志位，用于初始化，或者counterCells扩容时
    private transient volatile int cellsBusy;

    /**
     * Table of counter cells. When non-null, size is a power of 2.
     */
    // 【CHENG】：用于高并发的计数单元，如果初始化了这些计数单元，那么跟table数组一样，长度必须是2^n的形式
    private transient volatile CounterCell[] counterCells;
    
    @sun.misc.Contended static final class CounterCell {
        volatile long value;
        CounterCell(long x) { value = x; }
    }

    // views
    // 【CHENG】：一些不需要持久化的（类似于数据库视图）
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;
```

# 构造方法

## ConcurrentHashMap()
```java
    // 【CHENG】：默认构造方法，没啥好说的，就是个空方法
    public ConcurrentHashMap() {
    }
```

## ConcurrentHashMap(int initialCapacity)
```java
    // 【CHENG】：根据指定大小初始化
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   // 【CHENG】：又见到这个tableSizeFor（找比参数值大的第一个power of 2的值）
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        
        // 【CHENG】：把初始容量记下来（>0）
        this.sizeCtl = cap;
    }
```

## ConcurrentHashMap(Map<? extends K, ? extends V> m)
```java
    // 【CHENG】：通过别的Map构造的话，就是putAll的过程
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        // 【CHENG】：这时候就取默认容量
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }
```

## ConcurrentHashMap(int initialCapacity, float loadFactor)
```java
    // 【CHENG】：调的构造方法5
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }
```

## ConcurrentHashMap(...3params)
```java
    // 【CHENG】：带初始化大小，带负载因子，带预估的并发线程数
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```

# 初始化


## initTable
```java
    /**
     * Initializes table, using the size recorded in sizeCtl.
     */
    // 【CHENG】：根据sizeCtl来初始化table
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
        
            // 【CHENG】：sizeCtl<0 表示有线程正在进行初始化操作，把线程挂起；因为初始化只能有一个线程操作
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            
            // 【CHENG】：SIZECTL表示sizeCtl变量在内存中的偏移量（U.objectFieldOffset(k.getDeclaredField("sizeCtl"));）
            // 【CHENG】：还是CAS，若SIZECTL==sc（当前的sizeCtl），就让SIZECTL=-1，返回true（表示本线程正在进行初始化）；否则就不操作返回false
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    // 【CHENG】：再判断一次table是不是为空（类似于单例模式的双重判断）
                    if ((tab = table) == null || tab.length == 0) {
                        
                        // 【CHENG】：sc>0就代表threshold；否则取默认容量
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        
                        // 【CHENG】：直接初始化一个Node数组（大小已确定）
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        
                        // 【CHENG】：赋值给table咯
                        table = tab = nt;
                        
                        // 【CHENG】： sc = n - n/4 = 0.75n（也就是下一个扩容阈值）
                        sc = n - (n >>> 2);
                    }
                } finally {
                    // 【CHENG】：更新sizeCtl
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

# 增

## put
```java
    /**
     * 【CHENG】：和HashMap雷同
     */
    public V put(K key, V value) {
        return putVal(key, value, false);
    }
    
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        
        // 【CHENG】：基本的参数校验，一言不合就空指针异常了
        if (key == null || value == null) throw new NullPointerException();
        
        // 【CHENG】：计算本key的hash的方法，具体怎么算的，暂时还没研究
        int hash = spread(key.hashCode());
        
        // 【CHENG】：计算该链表节点的数量
        int binCount = 0;
        
        // 【CHENG】：遍历table数组（死循环的，肯定是内部判断完成break了）
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            
            // 【CHENG】：发现table是空的，那就初始化一下（第一次 put 操作的时候）
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            
            // 【CHENG】：如果table已经初始化了，看看本hash所在的位置是不是为空，为空的话进入此分支
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                
                // 【CHENG】：这里表示如果tab[i]==null，那么就让tab[i]=new Node<K,V>(hash, key, value, null)；否则tab[i]还是继续=null
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                    
                    // 【CHENG】：既然Node都塞进table里面了，自然就结束这个for循环了；
                    break;                   // no lock when adding to empty bin
            }
            
            // 【CHENG】：如果 hash 冲突了（也就是对应的table这个index下已经有Node了）
            // 【CHENG】：这时 hash 值还为 -1，说明是 ForwardingNode 对象（这是一个占位符对象，保存了扩容后的容器），表示正在扩容
            else if ((fh = f.hash) == MOVED)
            
                // 【CHENG】：这时实际是正在扩容，那么就帮助其扩容，以加快速度（整合表）
                tab = helpTransfer(tab, f);
            
            // 【CHENG】：如果 hash 冲突了但是hash不等于-1（就是正常塞数据的流程了）
            else {
                V oldVal = null;
                
                // 【CHENG】：锁的是其中一个bucket（也就是锁住了table[i]这个链表/树的头结点），而不是整个table
                synchronized (f) {
                    
                    // 【CHENG】：本hash所落的位置正好在当前锁的这个bucket上
                    if (tabAt(tab, i) == f) {
                    
                        // 【CHENG】：fh > 0 说明这个节点是一个链表的节点 不是树的节点（为什么？）
                        if (fh >= 0) {                            
                            
                            binCount = 1;
                            
                            // 【CHENG】：遍历链表的所有节点并计数
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                
                                // 【CHENG】：命中了相同的key
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                     
                                    // 【CHENG】：需要根据onlyIfAbsent确定是否替换value
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    
                                    // 【CHENG】：既然都命中了，循环就结束了
                                    break;
                                }
                                
                                // 【CHENG】：跟hashmap类似，找到最后一个节点（next为null），就把这个新Node追加到它的next
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key, value, null);
                                    break;
                                }
                                
                                // 【CHENG】：突发奇想，如果一直没有找到(e = e.next) == null呢？（只有一种可能就是Node自循环了）
                            }
                        }
                        
                        // 【CHENG】：这个分支就走到树的逻辑了
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            
                            // 【CHENG】：树的话直接设置为2，不会引发扩容
                            binCount = 2;
                            
                            // 【CHENG】：putTreeVal如果存在相同key的节点，就返回这个节点了；如果不存在，返回为空就不会走到内部去了
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                
                if (binCount != 0) {
                
                    // 【CHENG】：如果链表节点数达到了转树的限制了就转了
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    
                    // 【CHENG】：有老的值就返回老的值
                    if (oldVal != null)
                        return oldVal;
                    
                    // 【CHENG】：原来不存在值，就跳出处理逻辑了
                    break;
                }
            }
        }
        
        // 【CHENG】：将当前ConcurrentHashMap的元素数量+1，table的扩容是在这里发生的（方法内部逻辑还比较复杂，后述）
        addCount(1L, binCount);
        return null;
    }
    
    // 【Reference 01】：计算hash的方法
    static final int HASH_BITS = 0x7fffffff; 
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
    
    // 【Reference 02】：取某个Node的方法（这里比较特殊用到了Unsafe，Unsafe类提供了硬件级别的原子操作，JUC里很多地方也用到了）
    private static final sun.misc.Unsafe U;
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
    
    // 【Reference 03】：值交换（CAS操作）
    // 【CHENG】：tab[i]和c比较，如果相等就tab[i]=v，否则tab[i]=c;
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
```

# 删

## remove
```java
    /**
     * 【CHENG】：和HashMap雷同
     */
```

# 改
# 查

# 扩容

## helpTransfer
```java
    /**
     * Helps transfer if a resize is in progress.
     */
    // 【CHENG】：正在扩容时过来协助一下
    final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        
        // 【CHENG】：必须要保证当前节点是ForwardingNode，而且它的nextTable也不是空的，才可以加入协助
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            
            // 【CHENG】：根据 length 得到一个标识符号（这里的计算方式和sizeCtl还是有一定渊源的）
            int rs = resizeStamp(tab.length);
            
            // 【CHENG】：如果nextTab（新table）暂时还没有被并发修改；且tab（当前的table）也没有被并发修改；且sizeCtl  < 0（有线程在扩容）
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
                
                // 【CHENG】：以下任一场景都不会继续协助扩容；强势break
                // 【CHENG】：sc >>> 16（如果这个值和前面的rs不等，表示扩容标识变化了）
                // 【CHENG】：sizeCtl == rs + 1 （扩容结束了，不再有线程进行扩容；因为一旦有线程结束扩容，就会将 sc 减一[参考：addCount ]）
                // 【CHENG】：sizeCtl == rs + 65535（线程数太大了）
                // 【CHENG】：transferIndex <= 0（表示转移下标正在调整，也就是扩容结束了）
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    
                    // 【CHENG】：直接Over
                    break;
                
                // 【CHENG】：如果以上条件都没满足, 将sizeCtl + 1（表示增加了一个扩容线程）
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    
                    // 【CHENG】：扩容方法的真容
                    transfer(tab, nextTab);
                    
                    // 【CHENG】：扩完结束
                    break;
                }
            }
            
            // 【CHENG】：都扩了，肯定要把新的table返回了
            return nextTab;
        }
        
        // 【CHENG】：协助失败，那就返回当前的table吧
        return table;
    }
```

## transfer
```java
    /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    // 【CHENG】：方法行数有点多，看起来有点怵
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        
        // 【CHENG】：一些变量定义
        int n = tab.length, stride;
        
        // 【CHENG】：将length除以8然后除以CPU核心数；如果结果小于16，那么就用16。
        // 【CHENG】：n >>> 3 实例 0010 0000 -> 0000 0100 （32->4）
        // 【CHENG】：所以在tab.length比较小的情况下，不会有多线程介入
        // 【CHENG】：这么计算是为了让每个 CPU 处理的bucket一样多
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        
        // 【CHENG】：还没有初始化的新table
        if (nextTab == null) {            // initiating
            try {
                
                // 【CHENG】：创建一个新的数组，并赋给nextTab了
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                
                // 【CHENG】：如果出现异常的话，直接把数组容量置位最大然后结束扩容了
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            
            // 【CHENG】：给成员变量赋值
            nextTable = nextTab;
            
            // 【CHENG】：转移下标赋值（老table的length）
            transferIndex = n;
        }
        
        // 【CHENG】：取新table的length
        int nextn = nextTab.length;
        
        // 【CHENG】：构造一个ForwardingNode（这个构造方法里面就会设置hash=MOVED，也就和前面put那边呼应了）
        // 【CHENG】：这个变量主要用途是占位，当别的线程发现这个槽位中是 ForwardingNode，就会跳过这个节点了
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        
        // 【CHENG】：首次推进为 true，如果等于 true，说明需要再次推进一个下标（i--），反之，如果是 false，那么就不能推进下标，需要将当前的下标处理完毕才能继续推进
        boolean advance = true;
        
        // 【CHENG】：扩容完成状态
        boolean finishing = false; // to ensure sweep before committing nextTab
        
        // 【CHENG】：又是死循环,i 表示下标，bound 表示当前线程可以处理的当前桶区间最小下标
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            
            // 【CHENG】：如果当前线程可以继续往后处理
            while (advance) {
                int nextIndex, nextBound;
                
                // 【CHENG】：--i >= bound表示已经没有待移动的Node了
                // 【CHENG】：finishing在整个while循环里面都没有任何操作，这里永远是false吧？
                if (--i >= bound || finishing)
                    // 【CHENG】：下一次判断就会结束循环了
                    advance = false;
                
                // 【CHENG】：
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                
                // 【CHENG】：
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```
    
## addCount
```java
    /**
     * 【CHENG】：和HashMap雷同
     */
    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                if (sc < 0) {
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }

```

## put
```java
    /**
     * 【CHENG】：和HashMap雷同
     */
```

# 其他方法

## spread
```java
    // 【CHENG】：hash扰动函数，跟1.8的HashMap的基本一样，& HASH_BITS用于把hash值转化为正数，负数hash是有特别的作用的
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```

## tableSizeFor
```java
    // 【CHENG】：用于求2^n，用来作为table数组的容量，同1.8的HashMap
    private static final int tableSizeFor(int c) {
        int n = c - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

## comparableClassFor
```java
    // 【CHENG】：1.8的HashMap中讲解红黑树相关的时候说过，用于获取Comparable接口中的泛型类
    static Class<?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {
            Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
            if ((c = x.getClass()) == String.class) // bypass checks
                return c;
            if ((ts = c.getGenericInterfaces()) != null) {
                for (int i = 0; i < ts.length; ++i) {
                    if (((t = ts[i]) instanceof ParameterizedType) &&
                        ((p = (ParameterizedType)t).getRawType() == Comparable.class) &&
                        (as = p.getActualTypeArguments()) != null &&
                        as.length == 1 && as[0] == c) // type arg is c
                        return c;
                }
            }
        }
        return null;
    }
```

## compareComparables
```java
    // 【CHENG】：同1.8的HashMap，当类型相同且实现Comparable时，调用compareTo比较大小
    @SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
    static int compareComparables(Class<?> kc, Object k, Object x) {
        return (x == null || x.getClass() != kc ? 0 :  ((Comparable)k).compareTo(x)); 
    }

```

## from Unsafe
```java
    // 【CHENG】：下面几个用于读写table数组，使用Unsafe提供的更强的功能（数组元素的volatile读写，CAS 更新）代替普通的读写，调用者预先进行参数控制
     
    // 【CHENG】：volatile读取table[i]
    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
     
    // 【CHENG】：CAS更新table[i]，也就是Node链表的头节点，或者TreeBin节点（它持有红黑树的根节点）
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
     
    // 【CHENG】：volatile写入table[i]
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```

## treeifyBin
```java
    // 【CHENG】：满足变换为红黑树的两个条件时（链表长度这个条件调用者保证，这里只验证Map容量这个条件），将链表变为红黑树，否则只是进行一次扩容操作
    private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY) // Map的容量不够时，只是进行一次扩容
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) {
                    if (tabAt(tab, index) == b) {
                        TreeNode<K,V> hd = null, tl = null;
                        for (Node<K,V> e = b; e != null; e = e.next) {
                            TreeNode<K,V> p = new TreeNode<K,V>(e.hash, e.key, e.val, null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
                        setTabAt(tab, index, new TreeBin<K,V>(hd));
                    }
                }
            }
        }
    }
```

## untreeify
```java
    // 【CHENG】：规模不足时把红黑树转化为链表，此方法由调用者进行synchronized加锁，所以这里不加锁
    static <K,V> Node<K,V> untreeify(Node<K,V> b) {
        Node<K,V> hd = null, tl = null;
        for (Node<K,V> q = b; q != null; q = q.next) {
            Node<K,V> p = new Node<K,V>(q.hash, q.key, q.val, null);
            if (tl == null)
                hd = p;
            else
                tl.next = p;
            tl = p;
        }
        return hd;
    }
```

# 内部类

## Node
```java
    /**
     * 【CHENG】：Key-value entry.和HashMap的结构还是基本一样的（但是有个重点是加了volatile关键字）
     */
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }
```

## TreeNode
```java
    /**
     * Nodes for use in TreeBins
     */
    static final class TreeNode<K,V> extends Node<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;

        TreeNode(int hash, K key, V val, Node<K,V> next,
                 TreeNode<K,V> parent) {
            super(hash, key, val, next);
            this.parent = parent;
        }

        Node<K,V> find(int h, Object k) {
            return findTreeNode(h, k, null);
        }

        /**
         * Returns the TreeNode (or null if not found) for the given key
         * starting at given root.
         */
        final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
            if (k != null) {
                TreeNode<K,V> p = this;
                do  {
                    int ph, dir; K pk; TreeNode<K,V> q;
                    TreeNode<K,V> pl = p.left, pr = p.right;
                    if ((ph = p.hash) > h)
                        p = pl;
                    else if (ph < h)
                        p = pr;
                    else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                        return p;
                    else if (pl == null)
                        p = pr;
                    else if (pr == null)
                        p = pl;
                    else if ((kc != null ||
                              (kc = comparableClassFor(k)) != null) &&
                             (dir = compareComparables(kc, k, pk)) != 0)
                        p = (dir < 0) ? pl : pr;
                    else if ((q = pr.findTreeNode(h, k, kc)) != null)
                        return q;
                    else
                        p = pl;
                } while (p != null);
            }
            return null;
        }
    }
```

## ForwardingNode
```java
    /**
     * A node inserted at head of bins during transfer operations.
     */
    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }

        Node<K,V> find(int h, Object k) {
            // loop to avoid arbitrarily deep recursion on forwarding nodes
            outer: for (Node<K,V>[] tab = nextTable;;) {
                Node<K,V> e; int n;
                if (k == null || tab == null || (n = tab.length) == 0 ||
                    (e = tabAt(tab, (n - 1) & h)) == null)
                    return null;
                for (;;) {
                    int eh; K ek;
                    if ((eh = e.hash) == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                    if (eh < 0) {
                        if (e instanceof ForwardingNode) {
                            tab = ((ForwardingNode<K,V>)e).nextTable;
                            continue outer;
                        }
                        else
                            return e.find(h, k);
                    }
                    if ((e = e.next) == null)
                        return null;
                }
            }
        }
    }
```

## TreeBin
```java

    static final class TreeBin<K,V> extends Node<K,V> {
        TreeNode<K,V> root;
        volatile TreeNode<K,V> first;
        volatile Thread waiter;
        volatile int lockState;
        // values for lockState
        static final int WRITER = 1; // set while holding write lock
        static final int WAITER = 2; // set when waiting for write lock
        static final int READER = 4; // increment value for setting read lock

        /**
         * Tie-breaking utility for ordering insertions when equal
         * hashCodes and non-comparable. We don't require a total
         * order, just a consistent insertion rule to maintain
         * equivalence across rebalancings. Tie-breaking further than
         * necessary simplifies testing a bit.
         */
        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }

        /**
         * Creates bin with initial set of nodes headed by b.
         */
        TreeBin(TreeNode<K,V> b) {
            super(TREEBIN, null, null, null);
            this.first = b;
            TreeNode<K,V> r = null;
            for (TreeNode<K,V> x = b, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (r == null) {
                    x.parent = null;
                    x.red = false;
                    r = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = r;;) {
                        int dir, ph;
                        K pk = p.key;
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);
                            TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            r = balanceInsertion(r, x);
                            break;
                        }
                    }
                }
            }
            this.root = r;
            assert checkInvariants(root);
        }

        /**
         * Acquires write lock for tree restructuring.
         */
        private final void lockRoot() {
            if (!U.compareAndSwapInt(this, LOCKSTATE, 0, WRITER))
                contendedLock(); // offload to separate method
        }

        /**
         * Releases write lock for tree restructuring.
         */
        private final void unlockRoot() {
            lockState = 0;
        }

        /**
         * Possibly blocks awaiting root lock.
         */
        private final void contendedLock() {
            boolean waiting = false;
            for (int s;;) {
                if (((s = lockState) & ~WAITER) == 0) {
                    if (U.compareAndSwapInt(this, LOCKSTATE, s, WRITER)) {
                        if (waiting)
                            waiter = null;
                        return;
                    }
                }
                else if ((s & WAITER) == 0) {
                    if (U.compareAndSwapInt(this, LOCKSTATE, s, s | WAITER)) {
                        waiting = true;
                        waiter = Thread.currentThread();
                    }
                }
                else if (waiting)
                    LockSupport.park(this);
            }
        }

        /**
         * Returns matching node or null if none. Tries to search
         * using tree comparisons from root, but continues linear
         * search when lock not available.
         */
        final Node<K,V> find(int h, Object k) {
            if (k != null) {
                for (Node<K,V> e = first; e != null; ) {
                    int s; K ek;
                    if (((s = lockState) & (WAITER|WRITER)) != 0) {
                        if (e.hash == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                            return e;
                        e = e.next;
                    }
                    else if (U.compareAndSwapInt(this, LOCKSTATE, s,
                                                 s + READER)) {
                        TreeNode<K,V> r, p;
                        try {
                            p = ((r = root) == null ? null :
                                 r.findTreeNode(h, k, null));
                        } finally {
                            Thread w;
                            if (U.getAndAddInt(this, LOCKSTATE, -READER) ==
                                (READER|WAITER) && (w = waiter) != null)
                                LockSupport.unpark(w);
                        }
                        return p;
                    }
                }
            }
            return null;
        }

        /**
         * Finds or adds a node.
         * @return null if added
         */
        final TreeNode<K,V> putTreeVal(int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if (p == null) {
                    first = root = new TreeNode<K,V>(h, k, v, null, null);
                    break;
                }
                else if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.findTreeNode(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.findTreeNode(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    TreeNode<K,V> x, f = first;
                    first = x = new TreeNode<K,V>(h, k, v, f, xp);
                    if (f != null)
                        f.prev = x;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    if (!xp.red)
                        x.red = true;
                    else {
                        lockRoot();
                        try {
                            root = balanceInsertion(root, x);
                        } finally {
                            unlockRoot();
                        }
                    }
                    break;
                }
            }
            assert checkInvariants(root);
            return null;
        }

        /**
         * Removes the given node, that must be present before this
         * call.  This is messier than typical red-black deletion code
         * because we cannot swap the contents of an interior node
         * with a leaf successor that is pinned by "next" pointers
         * that are accessible independently of lock. So instead we
         * swap the tree linkages.
         *
         * @return true if now too small, so should be untreeified
         */
        final boolean removeTreeNode(TreeNode<K,V> p) {
            TreeNode<K,V> next = (TreeNode<K,V>)p.next;
            TreeNode<K,V> pred = p.prev;  // unlink traversal pointers
            TreeNode<K,V> r, rl;
            if (pred == null)
                first = next;
            else
                pred.next = next;
            if (next != null)
                next.prev = pred;
            if (first == null) {
                root = null;
                return true;
            }
            if ((r = root) == null || r.right == null || // too small
                (rl = r.left) == null || rl.left == null)
                return true;
            lockRoot();
            try {
                TreeNode<K,V> replacement;
                TreeNode<K,V> pl = p.left;
                TreeNode<K,V> pr = p.right;
                if (pl != null && pr != null) {
                    TreeNode<K,V> s = pr, sl;
                    while ((sl = s.left) != null) // find successor
                        s = sl;
                    boolean c = s.red; s.red = p.red; p.red = c; // swap colors
                    TreeNode<K,V> sr = s.right;
                    TreeNode<K,V> pp = p.parent;
                    if (s == pr) { // p was s's direct parent
                        p.parent = s;
                        s.right = p;
                    }
                    else {
                        TreeNode<K,V> sp = s.parent;
                        if ((p.parent = sp) != null) {
                            if (s == sp.left)
                                sp.left = p;
                            else
                                sp.right = p;
                        }
                        if ((s.right = pr) != null)
                            pr.parent = s;
                    }
                    p.left = null;
                    if ((p.right = sr) != null)
                        sr.parent = p;
                    if ((s.left = pl) != null)
                        pl.parent = s;
                    if ((s.parent = pp) == null)
                        r = s;
                    else if (p == pp.left)
                        pp.left = s;
                    else
                        pp.right = s;
                    if (sr != null)
                        replacement = sr;
                    else
                        replacement = p;
                }
                else if (pl != null)
                    replacement = pl;
                else if (pr != null)
                    replacement = pr;
                else
                    replacement = p;
                if (replacement != p) {
                    TreeNode<K,V> pp = replacement.parent = p.parent;
                    if (pp == null)
                        r = replacement;
                    else if (p == pp.left)
                        pp.left = replacement;
                    else
                        pp.right = replacement;
                    p.left = p.right = p.parent = null;
                }

                root = (p.red) ? r : balanceDeletion(r, replacement);

                if (p == replacement) {  // detach pointers
                    TreeNode<K,V> pp;
                    if ((pp = p.parent) != null) {
                        if (p == pp.left)
                            pp.left = null;
                        else if (p == pp.right)
                            pp.right = null;
                        p.parent = null;
                    }
                }
            } finally {
                unlockRoot();
            }
            assert checkInvariants(root);
            return false;
        }

        /* ------------------------------------------------------------ */
        // Red-black tree methods, all adapted from CLR

        static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                              TreeNode<K,V> p) {
            TreeNode<K,V> r, pp, rl;
            if (p != null && (r = p.right) != null) {
                if ((rl = p.right = r.left) != null)
                    rl.parent = p;
                if ((pp = r.parent = p.parent) == null)
                    (root = r).red = false;
                else if (pp.left == p)
                    pp.left = r;
                else
                    pp.right = r;
                r.left = p;
                p.parent = r;
            }
            return root;
        }

        static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                               TreeNode<K,V> p) {
            TreeNode<K,V> l, pp, lr;
            if (p != null && (l = p.left) != null) {
                if ((lr = p.left = l.right) != null)
                    lr.parent = p;
                if ((pp = l.parent = p.parent) == null)
                    (root = l).red = false;
                else if (pp.right == p)
                    pp.right = l;
                else
                    pp.left = l;
                l.right = p;
                p.parent = l;
            }
            return root;
        }

        static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                                    TreeNode<K,V> x) {
            x.red = true;
            for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
                if ((xp = x.parent) == null) {
                    x.red = false;
                    return x;
                }
                else if (!xp.red || (xpp = xp.parent) == null)
                    return root;
                if (xp == (xppl = xpp.left)) {
                    if ((xppr = xpp.right) != null && xppr.red) {
                        xppr.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.right) {
                            root = rotateLeft(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateRight(root, xpp);
                            }
                        }
                    }
                }
                else {
                    if (xppl != null && xppl.red) {
                        xppl.red = false;
                        xp.red = false;
                        xpp.red = true;
                        x = xpp;
                    }
                    else {
                        if (x == xp.left) {
                            root = rotateRight(root, x = xp);
                            xpp = (xp = x.parent) == null ? null : xp.parent;
                        }
                        if (xp != null) {
                            xp.red = false;
                            if (xpp != null) {
                                xpp.red = true;
                                root = rotateLeft(root, xpp);
                            }
                        }
                    }
                }
            }
        }

        static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root,
                                                   TreeNode<K,V> x) {
            for (TreeNode<K,V> xp, xpl, xpr;;)  {
                if (x == null || x == root)
                    return root;
                else if ((xp = x.parent) == null) {
                    x.red = false;
                    return x;
                }
                else if (x.red) {
                    x.red = false;
                    return root;
                }
                else if ((xpl = xp.left) == x) {
                    if ((xpr = xp.right) != null && xpr.red) {
                        xpr.red = false;
                        xp.red = true;
                        root = rotateLeft(root, xp);
                        xpr = (xp = x.parent) == null ? null : xp.right;
                    }
                    if (xpr == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpr.left, sr = xpr.right;
                        if ((sr == null || !sr.red) &&
                            (sl == null || !sl.red)) {
                            xpr.red = true;
                            x = xp;
                        }
                        else {
                            if (sr == null || !sr.red) {
                                if (sl != null)
                                    sl.red = false;
                                xpr.red = true;
                                root = rotateRight(root, xpr);
                                xpr = (xp = x.parent) == null ?
                                    null : xp.right;
                            }
                            if (xpr != null) {
                                xpr.red = (xp == null) ? false : xp.red;
                                if ((sr = xpr.right) != null)
                                    sr.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateLeft(root, xp);
                            }
                            x = root;
                        }
                    }
                }
                else { // symmetric
                    if (xpl != null && xpl.red) {
                        xpl.red = false;
                        xp.red = true;
                        root = rotateRight(root, xp);
                        xpl = (xp = x.parent) == null ? null : xp.left;
                    }
                    if (xpl == null)
                        x = xp;
                    else {
                        TreeNode<K,V> sl = xpl.left, sr = xpl.right;
                        if ((sl == null || !sl.red) &&
                            (sr == null || !sr.red)) {
                            xpl.red = true;
                            x = xp;
                        }
                        else {
                            if (sl == null || !sl.red) {
                                if (sr != null)
                                    sr.red = false;
                                xpl.red = true;
                                root = rotateLeft(root, xpl);
                                xpl = (xp = x.parent) == null ?
                                    null : xp.left;
                            }
                            if (xpl != null) {
                                xpl.red = (xp == null) ? false : xp.red;
                                if ((sl = xpl.left) != null)
                                    sl.red = false;
                            }
                            if (xp != null) {
                                xp.red = false;
                                root = rotateRight(root, xp);
                            }
                            x = root;
                        }
                    }
                }
            }
        }

        /**
         * Recursive invariant check
         */
        static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
            TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
                tb = t.prev, tn = (TreeNode<K,V>)t.next;
            if (tb != null && tb.next != t)
                return false;
            if (tn != null && tn.prev != t)
                return false;
            if (tp != null && t != tp.left && t != tp.right)
                return false;
            if (tl != null && (tl.parent != t || tl.hash > t.hash))
                return false;
            if (tr != null && (tr.parent != t || tr.hash < t.hash))
                return false;
            if (t.red && tl != null && tl.red && tr != null && tr.red)
                return false;
            if (tl != null && !checkInvariants(tl))
                return false;
            if (tr != null && !checkInvariants(tr))
                return false;
            return true;
        }

        private static final sun.misc.Unsafe U;
        private static final long LOCKSTATE;
        static {
            try {
                U = sun.misc.Unsafe.getUnsafe();
                Class<?> k = TreeBin.class;
                LOCKSTATE = U.objectFieldOffset
                    (k.getDeclaredField("lockState"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
    }
```

## ReservationNode
```java
    /**
     * A place-holder node used in computeIfAbsent and compute
     */
    static final class ReservationNode<K,V> extends Node<K,V> {
        ReservationNode() {
            super(RESERVED, null, null, null);
        }

        Node<K,V> find(int h, Object k) {
            return null;
        }
    }
```

# 关于sizeCtl 
&emsp;&emsp;在扩容操作中有个关键的变量sizeCtl，实际上该变量高 16 位保存 length 生成的标识符，低 16 位保存并发扩容的线程数
<img style="clear: both;display: block;margin:auto;" src="/img/2018/2018-08-22-07.jpg" width="60%">