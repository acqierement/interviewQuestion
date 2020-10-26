ReentrantLock 的用法

## ReentrantLock 的用法

### lock()

获取锁，需要相应地释放锁unlock()

### lockInterruptibly()

可以响应中断的获取锁的方法`lockInterruptibly()`

如果线程被中断interrupt()，那么会抛出异常。

### tryLock()

在调用时，该锁没有被其他线程获取的话，则获取到锁。

tryLock(long timeout, TimeUnit unit)：在给定的等待时间内没有被其他线程获取锁，且当前线程未被中断，则获取该锁。

## 实现原理

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

