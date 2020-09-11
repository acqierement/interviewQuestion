# lock ，sychronized，volatile的区别

https://blog.csdn.net/asdf717/article/details/47252763

https://blog.csdn.net/ztchun/article/details/60778950

[Java中的多线程你只要看这一篇就够了](https://www.cnblogs.com/wxd0108/p/5479442.html)

volatile https://www.cnblogs.com/dolphin0520/p/3920373.html

synchronize  https://www.cnblogs.com/dolphin0520/p/3923737.html

lock https://www.cnblogs.com/dolphin0520/p/3923167.html

## volatile

volatile 是java的一个关键字，只能修饰变量。它保证了变量的可见性，但是不能保证原子性。是一种比synchronize更轻量级的同步机制。

volatile修饰的变量不会进行重排序。

## synchronize

synchronize可以修饰方法和代码块。**是一个重入锁，而且是非公平锁**。具有可见性和原子性

当修饰方法时，获得的是对象锁，如果生成两个同样的对象，两个线程分别执行两个对象里的synchronize方法，不会发生阻塞。

如果一个对象有两个synchronize方法，那么如果两个线程分别调用同个对象的两个synchronize方法，由于获取的是对象锁，一个对象只有这么一个锁，所以有一个线程会阻塞。而如果一个调用的是synchronize方法，一个不是synchronize方法，则没有影响。

synchronize代码块比起修饰方法可以更精确的把需要的代码进行同步。synchronize（this）表示获得的是当前对象的锁，而如果括号里面是其他对象，表示获得的是其他对象的锁。

如果synchronize修饰的是静态方法，则获取的是这个类class的锁，等价于synchronize（Person.class),这个锁和对象锁不一样，彼此不会产生阻塞。



Lock和synchronize作用类似，但是lock可以实现更多的功能。lock要自己显示地释放锁，所以因此使用Lock时需要在finally块中释放锁。

- lock有带超时的获取锁尝试，

- 可以判断是否有线程，或者某个特定线程，在排队等待获取锁。

- tryLock()可以响应中断请求

- 还有条件变量（java.util.concurrent.Condition）可以实现更直观可控的同步操作

  

　　总结来说，Lock和synchronized有以下几点不同：

  1. Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

  2. synchronize是一个重入锁，而且是非公平锁，lock可以设置是否公平。

  3. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

  4. Lock有tryLock方法，可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

  5. 通过Lock的tryLock可以知道有没有成功获取锁，而synchronized却无法办到。

     > 在高版本JDK中，已经对synchronized进行了优化，synchronized和Lock方式在性能方面差别已不太明显





