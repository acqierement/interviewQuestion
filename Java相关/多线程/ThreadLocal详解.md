# ThreadLocal详解

## 数据结构

Thread类里面有threadLocals变量，它是一个ThreadLocal.ThreadLocalMap类型，我们放入的值就是存在这个map里面。

```java
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
```

既然是map，那就有key和value，来看一下ThreadLocalMap

```java

        /**
         * The initial capacity -- MUST be a power of two.
         */
        private static final int INITIAL_CAPACITY = 16;

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;

        /**
         * The number of entries in the table.
         */
        private int size = 0;

        /**
         * The next size value at which to resize.
         */
        private int threshold; // Default to 0
```

可以看到和基本的map差不多，主要是`private Entry[] table`这个变量。

来看一下entry。

```java
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```

可以看到继承了WeakReference弱引用，调用了WeakReference的构造函数，把key传了进去。所以ThreadLocal的key其实是个弱引用。

和HashMap不一样的是，entry没有next的引用，那个它是怎么解决hash冲突的呢？其实是使用线性探测的方式，如果当前hash值冲突了，就继续往后面找，知道找到一个为null的位置。具体可以看后面set的操作。

那key和value的值都是什么类型的呢？

ThreadLocal中set的源码里面有设置值。

```java
map.set(this, value);
```

value就是我们泛型里面定义的值，this就是当前的ThreadLocal对象，所以key就是ThreadLocal对象，值就是我们放进去的值。

## set()

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

getMap很简单，就是return t.threadLocals，获取线程的ThreadLocalMap。

然后set设置值。

如果map已经有值的话， 就直接set。

```java
        private void set(ThreadLocal<?> key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

可以看到通过nextIndex，一直往后面找。

```java
        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }
```

nextIndex其实是一个圈一样，一直在循环，知道找到key相同的，或者找到空位，再放置值。

这就是前面提到的线性探测。

如果map没有初始化，就进行创建。

```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
	// ThreadLocalMap构造函数
    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }
```



## get()

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

get其实就是从map获取值，也是挺简单的。注意到最后面有setInitialValue()，如果没有获取到值，才会到这一步。

```java
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

initialValue这个方法就是返回一个初始值。默认是返回null，子类可以继承他，重写该方法，当没有值的时候返回一个初始值，而不是null。

## InheritableThreadLocal

使用InheritableThreadLocal可以在子线程中取得父线程继承下来的值。

原理是在创建线程new Thread() 的时候，有这样的代码：

```java
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize) {
        init(g, target, name, stackSize, null, true);
    }
    
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
		......
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    }
```

可以看到如果inheritableThreadLocals有值的话，就会在子线程也创建inheritableThreadLocals。

```java
    static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }
```

其实就是将父线程的值一一复制到子线程的map中

```java
        private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for (int j = 0; j < len; j++) {
                Entry e = parentTable[j];
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                    if (key != null) {
                        // 可以重写改方法，去父线程的值进行处理
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }
```

InheritableThreadLocal继承ThreadLocal，里面只有三个自己的方法，getMap和createMap方法。

```java
    /**
     * 由于重写了getMap，操作InheritableThreadLocal时，
     * 将只影响Thread类中的inheritableThreadLocals变量，
     * 与threadLocals变量不再有关系
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * 类似于getMap，操作InheritableThreadLocal时，
     * 将只影响Thread类中的inheritableThreadLocals变量，
     * 与threadLocals变量不再有关系
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
```

还有一个childValue方法，子类可以重写这个方法，对父线程的值进行处理。

```java
    protected T childValue(T parentValue) {
        return parentValue;
    }
```

