# java多线程编程核心技术——总结

这本书很适合初学多线程的人，书中讲的都是一些基础多线程的概念，并结合代码展示。但是代码看多了就觉得烦了，有些代码其实只是说明了一些知识点。所以这里我就把代码去掉，把一些概念总结出来。

## 第一章 java多线程技能

## 1.2 使用多线程

### 1.2.1 继承Thread类

```java
package thread;

public class MyThread extends Thread {
    @Override
    public void run() {
        super.run();
        System.out.println("MyThread");
    }
}


```

运行类代码如下

```java
package thread;

public class RunDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
        System.out.println("运行结束");
    }
}
```

运行结果

```
运行结束
MyThread
```

这是一个简单的继承Thread的创建线程的方式。

这里主要是要说明**线程调用的随机性**，可以看到代码的运行结果与代码执行顺序或者调用顺序是无关。

打印运行结束的是main线程，而打印Mythread的是Mythread线程，虽然Mythread.start()在运行结束之前，但是由于main线程在之前就已经启动了，所以它执行“运行结束”就很快，而启动一个线程也需要一定的开销和耗时，所以Mythread打印就玩了。

还需要注意，执行start（）方法的顺序不代表线程启动的顺序。比如thread1.start（）；后面跟着thread2.start（），最终谁先执行是不确定的。

### 1.2.2 实现Runnable接口

```java
package thread;

public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("运行中");
    }
}
```



运行类代码

```java
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
    }
```

Thread类的构造函数中可以传入Runnable接口，不过Thread类也是实现了Runnable接口，所以也就意味着Thread的构造函数也可以传入Thread类的对象，这样做可以将一个Thread对象中的run方法交由其他的线程进行调用。

### 1.2.2 实现collable接口

待补充

### 1.2.3 实例变量与线程安全

自定义线程类中的实例变量针对其他线程可以有共享与不共享之分。在我们这个例子中，Mythread类中有一个成员变量 private int count = 5；在run方法中执行count--操作，直到减为0.

不共享的情况是这样的

```java
        MyThread a = new MyThread();
        MyThread b = new MyThread();
		......
```

创建了不同的MyThread实例，这样每个实例中都有自己的count值。

共享变量的情况是这样的

```java
        MyThread myThread = new MyThread();
        Thread a = new Thread(myThread);
        Thread b = new Thread(myThread);
		......
```

只创建了一个MyThread实例，然后将这个实例交给多个对象去执行，这样count变量就是共享的了，但是会存在多个线程访问到的值是一样的。出现了“非线程安全”的情况。因为在某些JVM中，i--的操作分为3步：

1. 取得原有i值。
2. 计算i-1。
3. 对i赋值。

在这三步中，如果有多线程同时访问，那么一定会出现“非线程安全”。

解决方法是在MyThread中的run方法加上synchronize的修饰符。

### 1.3 currentThread()方法

```
public static native Thread currentThread();
```

Thread.currentThread()方法返回一个Thread对象，可以知道代码段正在被哪个线程调用。

通过Thread.currentThread().getName()可以得到当前线程的名字，作者写了几段代码说明了执行myThread.start()，run方法是自动调用的，是被名称为Thread-0的线程调用的，而如果直接调用run方法，则是由main线程调用的。

### 1.4 isAlive()方法

isAlive()的功能是判断当前的线程是否处于活动状态。

什么是活动状态呢？活动状态就是线程已经启动且尚未终止。线程处于正在运行或准备开始运行的状态，就认为线程是“存活”的。

### 1.5 sleep()方法

sleep()方法的作用是在指定的毫秒数内让当前“正在执行的线程”休眠（暂停执行）。这个“正在执行的线程”就是this.currentThread()返回的线程。

### 1.6 getId()方法

getId()方法的作用是取得线程的唯一标识。

### 1.7 停止线程

java中有三种停止线程的方法：

1. 使用退出标志，是线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop方法强行停止线程，但是不推荐，stop和supend以及resume方法都是作废过期的方法，使用它们会产生不可预料的结果。
3. 使用interrupt方法中断线程。该方法并不是马上停止循环，而仅仅只是在当前线程中打了一个停止的标记，并不是真的停止线程。

#### 1.7.2 判断线程是否是停止状态

java的sdk中，Thread.java类中提供了两种方法判断线程是否是停止状态。

1. Thread.interrupted()：测试当前线程是否已经中断。
2. myThread.isInterrupted()：测试线程是否已经中断。

首先，可以看到interrupted方法是用类名调用的，所以interrupted是一个静态方法。下面是这两个方法的声明。

```java
public static boolean interrupted()
public boolean isInterrupted()
```

再看看两个方法的作用，不同的地方在于interrupted指的是“当前线程“，由于该方法是个静态方法，所以可以在各个地方通过Thread.interrupted()调用，如果你是在main方法中调用，则判断的是main的这个线程是否停止。

而isInterrupted()需要通过实例对象来调用，所以它判断的就是这个线程。比如我们创建了myThread的实例对象，通过myThread.isInterrupted()就是判断myThread这个线程是否停止。




## 第二章 对象及变量的并发访问

## 第三章 线程间通信

### 3.1.3 wait()

wait的作用是使当前执行代码的线程进行等待，wait()方法是Object类的方法。

在调用wait()之前，线程必须获得该对象的对象级别锁，即只能在同步方法和同步代码块中调用wait()方法。在执行wait()方法后，当前线程释放锁。如果调用wait()方法时没有持有适当的锁，则抛出IllegalMonitorStateException异常，该异常是RuntimeException的子类，所以不需要显式捕获。

## 第四章 lock的使用

## 第五章 定时器timer

## 第六章 单例模式与多线程

## 第七章 拾遗增补

