ConcurrentHashMap详解

## jdk1.7版本

### 和其他并发集合的区别

待完善

### 数据结构

```java
    /**
     * The segments, each of which is a specialized hash table.
     */
    final Segment<K,V>[] segments;
```

可以看到主要就是一个Segment数组，注释也写了，每个都是一个特殊的hash table。

来看一下Segment是什么东西。

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    	......
            /**
         * The per-segment table. Elements are accessed via
         * entryAt/setEntryAt providing volatile semantics.
         */
        transient volatile HashEntry<K,V>[] table;
    
        transient int threshold;

        final float loadFactor;
    	// 构造函数
        Segment(float lf, int threshold, HashEntry<K,V>[] tab) {
            this.loadFactor = lf;
            this.threshold = threshold;
            this.table = tab;
        }
  		......
    ｝
```

上面是部分代码，可以看到Segment继承了ReentrantLock，所以其实每个Segment就是一个锁。

里面存放着HashEntry数组，该变量用volatile修饰。HashEntry和hashmap的节点类似，也是一个链表的节点。

来看看具体的代码，可以看到和hashmap里面稍微不同的是，他的成员变量有用volatile修饰。

```java
    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;

        HashEntry(int hash, K key, V value, HashEntry<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        ......
    }
```

所以ConcurrentHashMap的数据结构差不多是下图这种样子的。

![数据结构](C:\Users\EDZ\AppData\Roaming\Typora\typora-user-images\1591060799397.png)

在构造的时候，Segment 的数量由所谓的 concurrentcyLevel 决定，默认是 16，也可以在相应构造函数直接指定。注意，Java 需要它是 2 的幂数值，如果输入是类似 15 这种非幂值，会被自动调整到 16 之类 2 的幂数值。

来看看源码，先从简单的get方法开始

### get()

```java
    public V get(Object key) {
        Segment<K,V> s; // manually integrate access methods to reduce overhead
        HashEntry<K,V>[] tab;
        int h = hash(key);
        long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
        // 通过unsafe获取Segment数组的元素
        if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
            (tab = s.table) != null) {
            // 还是通过unsafe获取HashEntry数组的元素
            for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                     (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
                 e != null; e = e.next) {
                K k;
                if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                    return e.value;
            }
        }
        return null;
    }
```

get的逻辑很简单，就是找到Segment对应下标的HashEntry数组，再找到HashEntry数组对应下标的链表头，再遍历链表获取数据。

这个获取数组中的数据是使用UNSAFE.getObjectVolatile(segments, u)，unsafe提供了像c语言的可以直接访问内存的能力。该方法可以获取对象的相应偏移量的数据。u就是计算好的一个偏移量，所以等同于segments[i]，只是效率更高。

### put()

```java
    public V put(K key, V value) {
        Segment<K,V> s;
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key);
        int j = (hash >>> segmentShift) & segmentMask;
        if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
             (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
            s = ensureSegment(j);
        return s.put(key, hash, value, false);
    }
```

而对于 put 操作，是以 Unsafe 调用方式，直接获取相应的 Segment，然后进行线程安全的 put 操作：

主要逻辑在Segment内部的put方法

```java

final V put(K key, int hash, V value, boolean onlyIfAbsent) {
            // scanAndLockForPut会去查找是否有key相同Node
            // 无论如何，确保获取锁
            HashEntry<K,V> node = tryLock() ? null :
                scanAndLockForPut(key, hash, value);
            V oldValue;
            try {
                HashEntry<K,V>[] tab = table;
                int index = (tab.length - 1) & hash;
                HashEntry<K,V> first = entryAt(tab, index);
                for (HashEntry<K,V> e = first;;) {
                    if (e != null) {
                        K k;
                        // 更新已有value...
                    }
                    else {
                        // 放置HashEntry到特定位置，如果超过阈值，进行rehash
                        // ...
                    }
                }
            } finally {
                unlock();
            }
            return oldValue;
        }

```

### size()

来看一下主要的代码，

```java
for (;;) {
    // 如果重试次数等于默认的2，就锁住所有的segment，来计算值
    if (retries++ == RETRIES_BEFORE_LOCK) {
        for (int j = 0; j < segments.length; ++j)
            ensureSegment(j).lock(); // force creation
    }
    sum = 0L;
    size = 0;
    overflow = false;
    for (int j = 0; j < segments.length; ++j) {
        Segment<K,V> seg = segmentAt(segments, j);
        if (seg != null) {
            sum += seg.modCount;
            int c = seg.count;
            if (c < 0 || (size += c) < 0)
                overflow = true;
        }
    }
    // 如果sum不再变化，就表示得到了一个确切的值
    if (sum == last)
        break;
    last = sum;
}
```

这里其实就是计算所有segment的数量和，如果数量和跟上次获取到的值相等，就表示map没有进行操作，这个值是相对正确的。如果重试两次之后还是没法得到一个统一的值，就锁住所有的segment，再来获取值。

### 扩容

```java
private void rehash(HashEntry<K,V> node) {

            HashEntry<K,V>[] oldTable = table;
            int oldCapacity = oldTable.length;
    		// 新表的大小是原来的两倍
            int newCapacity = oldCapacity << 1;
            threshold = (int)(newCapacity * loadFactor);
            HashEntry<K,V>[] newTable =
                (HashEntry<K,V>[]) new HashEntry[newCapacity];
            int sizeMask = newCapacity - 1;
            for (int i = 0; i < oldCapacity ; i++) {
                HashEntry<K,V> e = oldTable[i];
                if (e != null) {
                    HashEntry<K,V> next = e.next;
                    int idx = e.hash & sizeMask;
                    if (next == null)   //  Single node on list
                        newTable[idx] = e;
                    else { // Reuse consecutive sequence at same slot
                        // 如果有多个节点
                        HashEntry<K,V> lastRun = e;
                        int lastIdx = idx;
                        // 这里操作就是找到末尾的一段索引值都相同的链表节点，这段的头结点是lastRun.
                        for (HashEntry<K,V> last = next;
                             last != null;
                             last = last.next) {
                            int k = last.hash & sizeMask;
                            if (k != lastIdx) {
                                lastIdx = k;
                                lastRun = last;
                            }
                        }
                        // 然后将lastRun结点赋值给数组位置，这样lastRun后面的节点也跟着过去了。
                        newTable[lastIdx] = lastRun;
                        // 之后就是复制开头到lastRun之间的节点
                        // Clone remaining nodes
                        for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                            V v = p.value;
                            int h = p.hash;
                            int k = h & sizeMask;
                            HashEntry<K,V> n = newTable[k];
                            newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                        }
                    }
                }
            }
            int nodeIndex = node.hash & sizeMask; // add the new node
            node.setNext(newTable[nodeIndex]);
            newTable[nodeIndex] = node;
            table = newTable;
        }
```

## jdk1.8版本

### 数据结构

1.8的版本的ConcurrentHashmap整体上和Hashmap有点像，但是去除了segment，而是使用node的数组。

```java
    transient volatile Node<K,V>[] table;
```

1.8中还是有Segment这个内部类，但是存在的意义只是为了序列化兼容，实际已经不使用了。

来看一下node节点

```java
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
        ......
    }
```

和HashMap中的node节点类似，也是实现Map.Entry，不同的是val和next加上了volatile修饰来保证可见性。

### put()

```java

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                // 初始化
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                // 利用CAS去进行无锁线程安全操作，如果bin是空的
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                     // 细粒度的同步修改操作... 
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // 找到相同key就更新
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                // 没有相同的就新增
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        // 如果是树节点，进行树的操作
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                // Bin超过阈值，进行树化
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }

```

可以看到，在同步逻辑上，它使用的是 synchronized，而不是通常建议的 ReentrantLock 之类，这是为什么呢？现在 JDK1.8 中，synchronized 已经被不断优化，可以不再过分担心性能差异，另外，相比于 ReentrantLock，它可以减少内存消耗，这是个非常大的优势。

与此同时，更多细节实现通过使用 Unsafe 进行了优化，例如 tabAt 就是直接利用 getObjectAcquire，避免间接调用的开销。

那么，再来看看size是怎么操作的？

### size()

```java
    final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
```

这里就是获取成员变量counterCells，遍历获取总数。

其实，对于 CounterCell 的操作，是基于 java.util.concurrent.atomic.LongAdder 进行的，是一种 JVM 利用空间换取更高效率的方法，利用了Striped64内部的复杂逻辑。这个东西非常小众，大多数情况下，建议还是使用 AtomicLong，足以满足绝大部分应用的性能需求。

### 扩容

```java
 private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
		......
        // 初始化
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
     	// 是否继续处理下一个
        boolean advance = true;
     	// 是否完成
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                // 首次循环才会进来这里
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
                //扩容结束后做后续工作
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                //每当一条线程扩容结束就会更新一次 sizeCtl 的值，进行减 1 操作
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            // 如果是null，设置fwd
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            // 说明该位置已经被处理过了，不需要再处理
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                // 真正的处理逻辑
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
                        // 树节点操作
                        else if (f instanceof TreeBin) {
                            ......
                        }
                    }
                }
            }
        }
    }

```

核心逻辑和HashMap一样也是创建两个链表，只是多了获取lastRun的操作。