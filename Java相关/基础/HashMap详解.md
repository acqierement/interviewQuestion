HashMap详解

## 1.7的HashMap

首先来看一下它的数据结构

```java
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```

其实就是一个Entry数组

Entry的属性是这样的

```java
        final K key;
        V value;
        Entry<K,V> next;
        int hash;
		// 构造函数
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
```

所以其实可以看到Entry就是一个链表中的节点。

所以，我们对1.7的HashMap的数据结构就清楚了，首先是一个数组，然后每个数组里面的值是一个链表。就像下图

![1590655550907](HashMap详解/1590655550907.png)

想象一下put的过程，放进一个数的时候，根据hash值找到对应的数组下标位置，该下标的单元格放的是一个链表，我们在该链表上面加上一个节点。就完成了一个put操作。

具体来看一下源码是怎么实现的。

**put()**

```java
    public V put(K key, V value) {
        // 如果是空，就初始化
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        // 对null的key特别处理
        if (key == null)
            return putForNullKey(value);
        // 获取hash值
        int hash = hash(key);
        /****** 1. 根据hash值获取索引位置 *****/
        int i = indexFor(hash, table.length);
        // 遍历索引所在位置的链表
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // 如果hash值相等，并且key相等，就替换该值
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
		
        modCount++;
        /***** 2. 这个key之前都没出现过，就增加节点 *****/
        addEntry(hash, key, value, i);
        return null;
    }
```

其实这里面每个方法都有很多巧妙的地方，大家可以自己看看，我就主要讲一下有序号的那些方法。

根据hash值获取索引位置里面的代码很简单，return h & (length-1);在HashMap里面数组的大小确保了会是2的幂次方，所以length-1就是00001111这样的数，这样hash值异或就确保了最后的索引值会是小于length的正数。

然后是是增加节点的代码

**addEntry()**

```java
    void addEntry(int hash, K key, V value, int bucketIndex) {
        // 如果大小大于阈值，需要扩容
        if ((size >= threshold) && (null != table[bucketIndex])) {
            /***** 1.扩容的代码 *****/
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
		/***** 2.创建节点 *****/
        createEntry(hash, key, value, bucketIndex);
    }
```

1. 可以看到扩容的时候，是扩大了两倍，来看看代码

```java
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        // 确保大小不会超过最大值，最大值为1 << 30
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
		// 扩容只需要把数组扩大一下就好了
        Entry[] newTable = new Entry[newCapacity];
        // 但是麻烦的是，数组扩大之后，根据hash值获得的索引位置也会发生变化，
        // 所以需要把map里面的数据重新调整
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```