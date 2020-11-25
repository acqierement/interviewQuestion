## ReentrantLock 用法和实现原理

### lock()

获取锁，需要相应地释放锁unlock()

### lockInterruptibly()

可以响应中断的获取锁的方法`lockInterruptibly()`

如果线程被中断interrupt()，那么会抛出异常。

### tryLock()

在调用时，该锁没有被其他线程获取的话，则获取到锁。

tryLock(long timeout, TimeUnit unit)：在给定的等待时间内没有被其他线程获取锁，且当前线程未被中断，则获取该锁。

## 实现原理

ReentrantLock主要有两个构造函数，主要是为了生成公平锁还是非公平锁。

```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

lock都是调用sync的lock方法，所以公平和非公平的lock有各自的逻辑。

```java
    public void lock() {
        sync.lock();
    }
```

### 非公平锁

#### lock

```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

首先用一个CAS操作，判断state是否是0（表示当前锁未被占用），如果是0则把它置为1，并且设置当前线程为该锁的独占线程，表示获取锁成功。当多个线程同时尝试占用同一个锁时，CAS操作只能保证一个线程操作成功，剩下的只能乖乖的去排队啦。

　　“非公平”即体现在这里，如果占用锁的线程刚释放锁，state置为0，而排队等待锁的线程还未唤醒时，新来的线程就直接抢占了该锁，那么就“插队”了。

如果当前已经线程已经被抢占了，就进入到else里面的 acquire(1)中，其实公平锁也是同样的方法，只是内部的方法实现不一样。

```java
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

第一步尝试去获取锁，如果尝试获取锁成功，方法直接返回。tryAcquire公平锁和非公平锁都有自己的实现，虽然各自实现的方法名不一样，但是代码都是一样的，所以可以理解tryAcquire两种锁都是一样的。咱们就那非公平锁来看看：**nonfairTryAcquire**

```java
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

前半部分又再检查了一次State是否变成0了，也就是锁是否被释放了，如果被释放了，就获取锁。若不为0，则判读一下是否是被自己占用，如果是的话，也可以获取锁，然后更新state的值，表示重入的次数。

如果tryAcquire(arg)为false的话，表示获取锁不成功。那么就要将线程入队。

```java
    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        // 如果尾巴不为null，就在尾部加一个
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        // 如果尾巴为null，就初始化节点
        enq(node);
        return node;
    }
```

```java
    // enq里面主要使用自旋和CAS来实现并发操作
   private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                // 初始化队列
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

可以看到一直在自旋，如果队列没有初始化，就建一个head节点初始化队列。然后之后的自旋就会把节点加到队列后面。

之后会执行acquireQueued(final Node node, int arg)。这个方法让已经入队的线程尝试获取锁，若失败则会被挂起。

```java
/**
 * 已经入队的线程尝试获取锁
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true; //标记是否成功获取锁
    try {
        boolean interrupted = false; //标记线程是否被中断过
        for (;;) {
            final Node p = node.predecessor(); //获取前驱节点
            //如果前驱是head,即该结点已成老二，那么便有资格去尝试获取锁
            if (p == head && tryAcquire(arg)) {
                setHead(node); // 获取成功,将当前节点设置为head节点
                p.next = null;// help GC // 原head节点出队,在某个时间点被GC回收
                failed = false; //获取成功
                return interrupted; //返回是否被中断过
            }
            // 判断获取失败后是否可以挂起,若可以则挂起
            if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                // 线程若被中断,设置interrupted为true
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

**shouldParkAfterFailedAcquire**

```java
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        // 前驱节点状态为signal,返回true
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        //前驱节点状态为CANCELLED,从后往前把状态为CANCELLED的节点删除
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
             // 设置前驱节点的状态为SIGNAL
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

线程入队后能够挂起的前提是，它的前驱节点的状态为SIGNAL，它的含义是“Hi，前面的兄弟，如果你获取锁并且出队后，记得把我唤醒！”。所以shouldParkAfterFailedAcquire会先判断当前节点的前驱是否状态符合要求，若符合则返回true，然后调用parkAndCheckInterrupt，将自己挂起。如果不符合，再看前驱节点是否>0(CANCELLED)，若是则从后往前把cancel的节点删除，若不是则将前驱节点的状态设置为SIGNAL。

​     整个流程中，如果前驱结点的状态不是SIGNAL，那么自己就不能安心挂起，需要去找个安心的挂起点，同时可以再尝试下看有没有机会去尝试竞争锁。

#### unlock

再看一下unLock

```java
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

流程大致为先尝试释放锁，若释放成功，那么查看头结点的状态是否不等于0，如果是则唤醒头结点的下个节点关联的线程，如果释放失败那么返回false表示解锁失败。这里我们也发现了，每次都只唤起头结点后第一个不是取消的节点。

```java
        protected final boolean tryRelease(int releases) {
            // 获取加锁次数
            int c = getState() - releases;
            // 如果获取锁的线程不对，就抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            // 如果加锁次数为0，则释放锁
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            // 更新状态
            setState(c);
            return free;
        }
```

### 公平锁

#### lock

```java
        final void lock() {
            acquire(1);
        }
```

复习一下非公平锁的lock

```java
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }
```

公平锁的acquire和非公平锁其实就是一样的。所以就不需要再说了。

### tryLock

```java
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
	public final boolean tryAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        return tryAcquire(arg) ||
            doAcquireNanos(arg, nanosTimeout);
    }
```

主要看一下**doAcquireNanos**

```java
    private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (nanosTimeout <= 0L)
            return false;
        // 获取截止时间
        final long deadline = System.nanoTime() + nanosTimeout;
        // 加入队列
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                // 获取到锁就返回
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
                // 重新计算超时时间
                nanosTimeout = deadline - System.nanoTime();
                // 如果超时，获取锁失败
                if (nanosTimeout <= 0L)
                    return false;
                // 判断是否能够挂起，如果前驱节点会通知它，就可以挂起了
                if (shouldParkAfterFailedAcquire(p, node) &&
                    // 超时时间要足够长，不长的话就没必要挂起了
                    nanosTimeout > spinForTimeoutThreshold)
                    LockSupport.parkNanos(this, nanosTimeout);
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

doAcquireNanos的流程简述为：线程先入等待队列，然后开始自旋，尝试获取锁，获取成功就返回，失败则在队列里找一个安全点把自己挂起直到超时时间过期。

这里为什么还需要循环呢？因为当前线程节点的前驱状态可能不是SIGNAL，那么在当前这一轮循环中线程不会被挂起，然后更新超时时间，开始新一轮的尝试。









参考[ReentrantLock原理](https://blog.csdn.net/fuyuwei2015/article/details/83719444)