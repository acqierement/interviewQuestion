# java基础

## Map集合

### map了解吗，说说hashmap，hashtable，treemap

**关于null值**

只有HashMap支持null的key和value。

TreeMap支持null的value。因为要对key做比较，所以key不能为null。

HashTable 不支持null的key和value。对null的value做了限制，对key会获取key的hashcode，所以null的key会报错。

**线程安全**

HashTable是线程安全的，相关操作的方法都加了synchronized。

hashmap1.8阈值为什么是8

Hashtable 和 HashMap的区别 ： { 底层数据结构 (JDK1.8后不同)、父类不同  、扩容方法不同 、 线程上锁范围不同（重点） 红黑树区别，有序无序}

区别，性能，原理，hashmap为什么不安全，举例子，jdk1.7，jdk1.8区别；

哪些集合可以存放`null`？上述集合除了`Hashtable`和`ConcurrentHashMap`都可以。

哪些集合可以存放重复的元素？`Set`不能重复，`Map`的`key`不能重复。

hashmap：

Java 7及以前是数组+链表、Java 8及以后是数组+链表+红黑树、Java 8红黑树转变的条件、如何减少hash碰撞、数组的大小和如何用位运算代替取模、Java 7多线程扩容死循环问题、Java 8扩容时如何减少移动提高性能  

### `Hashtable`和`ConcurrentHashMap`差异？

- Hashtable`是旧版本Java的遗留类，继承自另一个遗留类`Dictionary`，线程安全的实现方式为简单粗暴地给`get`和`put`等方法加上`synchronized`关键字
- `Collections.synchronizedMap`方法，使用装饰者模式来给`get`和`put`等方法加上`synchronized`语句块，由于即使不考虑`Hashtable`的线程安全开销，`Hashtable`的实现性能也低于`HashMap`，因此使用此方法装饰`HashMap`会比直接使用`Hashtable`更为高效
- `ConcurrentHashMap`是`java.util.concurrent`包里的线程安全`Map`，Java 7及以前为分段+数组+链表，在每个段中使用可重入锁；Java 8为数组+链表+红黑树，在数组的每个位置使用CAS来进行轻量级锁

### ConcurrentHashMap

扩容？

put的源码

### HashMap & ConcurrentHashMap 的比较

CurconrentHashMap它的size()函数怎么保证返回正确值。是否有更好的方法可以更加去优化它的锁机制（没有答出来）

线程安全的CurconrentHashMap，1.7与1.8，中间因为说了volitale，又转到volitale，底层怎么实现的，解决什么问题

深入一些 ： HashMap 为什么线程不安全？ 能否举例 = { 并发resize()触发闭环结构 ，覆盖put操作 }

### HashMap

HashMap的底层实现

那是否可以一直往map中存入数据？

如果你4G的内存，让你设计一个Map存入数据，你会从哪几方面考虑

HashMap resize()过程能否介绍 ？   

HashMap效率受什么影响 (负载因子、hash数组size)？   

HashMap中扰动函数的作用 ？

负载因子过大，过小会怎么样？

说了红黑树然后又问二叉树和红黑树的底层实现

为什么用红黑树，红黑树和普通的平衡二叉树区别

什么时候转树，什么时候退树，remove后进行什么操作

为什么转树是8个，退树是6个？

hash方法为什么高16异或低16

为什么1.8引入红黑树，为什么阈值是8，红黑树的特点，时间复杂度，与其他数据结构的比较，为什么每次扩容二倍

hashmap需要存4个数，那么最少需要几个空间，从装载因子方面

## List集合

### ArrayList和LinkedList的实现与区别

ArrayList底层是Object[]数组，每次扩容时增加一半

```java
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

移动数组元素是使用System.arraycopy来操作的。



**LinkedList是单向链表还是双向链表**

LinkedList底层的数据结构是这样的

```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

可以看到是一个双向链表。插入数据就是一个双向链表的插入操作，没什么特别的。



## Set集合

### Set 了解过吗？知道 add（） 会出什么问题吗？

这题就是判断你是否是背题，没看到源码的很难说出来）

### `HashSet`了解吗？

根据`HashSet`是否有序，底层使用`HashMap`或者`LinkedHashMap`
根据是否有序，以`Set`中的元素作为`Map`中的`key`，以固定的`Object`对象作为`Map`中的`value`

如何保证Set中的元素不重复？
重写对象的equals和hashCode方法
解决哈希冲突的方法？
线性探查法、平分探查法、拉链法
常用的哈希算法？
MD5、SHA-1、SHA-256

### TreeSet和HashSet区别

## 队列

### 队列呢，了解什么

ArrayBlockQueue，LinkedBlockingQueue

### 各个队列使用场景

### queue里面都有什么方法

offer，add，put

### synchroBlockQueue 到底可以存几个

## 锁

### Synchronized

#### Synchronized的原理

#### sychronized、jdk1.6的优化，重量级锁自旋次数设置的参数

有sychronized、为什么引入ReenTrantLock

### 锁用过没

叭叭各种锁

### 锁的什么方法你用过

tryLock

getHoldCount

### lock和tryLock区别

## 异常

### 运行时异常和非运行时异常



RuntimeException Exception Error

### java哪些异常类需要try  catch   手动捕获

## 序列化

### 序列化与反序列化

### 序列化的知识点

### serializable关键字的作用（实现原理）

### 集中序列化协议,pb的优点

Protobuf 是 Google 公司内部的混合语言数据标准，是一种轻便、高效的结构化数据存储格式，可以用于结构化数据序列化，支持 Java、Python、C++、Go 等语言。Protobuf使用的时候需要定义 IDL（Interface description language），然后使用不同语言的 IDL编译器，生成序列化工具类，它的优点是：

- 序列化后体积相比 JSON、Hessian 小很多；

- IDL 能清晰地描述语义，所以足以帮助并保证应用程序之间的类型不会丢失，无需类似XML 解析器；
- 序列化反序列化速度很快，不需要通过反射获取类型；
- 消息格式升级和兼容性不错，可以做到向后兼容。

Protobuf 非常高效，但是对于具有反射和动态能力的语言来说，这样用起来很费劲，这一点就不如 Hessian

## 网络IO模型

### 五种常见的网络IO模型

### socket有几种状态

### NIO的实现原理

### BIO、NIO、AIO的对应类实现了解吗

### NIO的使用

### 多路io复用的几种实现

### NIO和BIO的区别

##  equals 和 == 区别

==本质上比较是内存地址，如果相等，则说明这两个引用实际是指向同一个对象地址的。

Object类中的equals也是使用==来判断，像String的equals比较的是char数组的各个元素是否相等。

为啥重写equals要重写hashCode()

hash值相等，而两个对象不一定equals

## String StringBuffer StringBuilder  区别 和各自使用场景,实现

深入一些 ： String 是如何实现它不可变的？ 为什么要设置String为不可变对象  ?     (字节一面这个问题给我问懵了)

StringBuilder为什么快

## 接口和抽象类区别

接口中的成员变量，隐式使用`public static final`修饰，方法隐式使用`public abstract`修饰

Java 8 增加了函数式编程的支持，所以又增加了一类定义，即所谓 functional interface，简单说就是只有一个抽象方法的接口，通常建议使用 @FunctionalInterface Annotation 来标记

从 Java 8 开始，interface 增加了对 default method 的支持，接口也可以有方法实现了。

抽象类中可以有非抽象的方法。

**能不能用一个设计模式来说明**

## 重写和重载的区别

**重写（Override）**

重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。**即外壳不变，核心重写！**

重写方法不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常

**重载(Overload)**

重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。

每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。

最常用的地方就是构造器的重载。

## 深拷贝和浅拷贝区别

## 说一下你理解的封装，继承和多态

**封装**的目的是隐藏事务内部的实现细节，以便提高安全性和简化编程。封装提供了合理的边界，避免外部调用者接触到内部的细节。我们在日常开发中，因为无意间暴露了细节导致的难缠 bug 太多了，比如在多线程环境暴露内部状态，导致的并发修改问题。从另外一个角度看，封装这种隐藏，也提供了简化的界面，避免太多无意义的细节浪费调用者的精力。

**继承**是代码复用的基础机制，类似于我们对于马、白马、黑马的归纳总结。但要注意，继承可以看作是非常紧耦合的一种关系，父类代码修改，子类行为也会变动。在实践中，过度滥用继承，可能会起到反效果。

**多态**，你可能立即会想到重写（override）和重载（overload）、向上转型。简单说，重写是父子类中相同名字和参数的方法，不同的实现；重载则是相同名字的方法，但是不同的参数，

##  Object的方法 

 { finalize 、 clone、 getClass 、 equals 、 hashCode 、toString、notify、notifyAll、wait}

## int的长度

byte：一个字节（8位）（-128~127）（-2的7次方到2的7次方-1）

short：两个字节（16位）（-32768~32767）（-2的15次方到2的15次方-1）

int：四个字节（32位）（一个字长）（-2147483648~2147483647）（-2的31次方到2的31次方-1）

long：八个字节（64位）（-9223372036854774808~9223372036854774807）（-2的63次方到2的63次方-1）

float：四个字节（32位）（3.402823e+38 ~ 1.401298e-45）（e+38是乘以10的38次方，e-45是乘以10的
负45次方）

double：八个字节（64位）（1.797693e+308~ 4.9000000e-324）

## int integer区别

（内存位置）

在 Java 5 中新增了静态工厂方法 valueOf，在调用它的时候会利用一个缓存机制，带来了明显的性能改进。按照 Javadoc，这个值默认缓存是 -128 到 127 之间。

这种缓存机制并不是只有 Integer 才有，同样存在于其他的一些包装类，比如：

Boolean，缓存了 true/false 对应实例，确切说，只会返回两个常量实例 Boolean.TRUE/FALSE。

Short，同样是缓存了 -128 到 127 之间的数值。

Byte，数值有限，所以全部都被缓存。

Character，缓存范围’\u0000’ 到 ‘\u007F’。

**自动装箱的时候做了哪些工作**

自动装箱实际上算是一种语法糖。什么是语法糖？可以简单理解为 Java 平台为我们自动进行了一些转换，保证不同的写法在运行时等价，它们发生在编译阶段，也就是生成的字节码是一致的。

javac 替我们自动把装箱转换为 Integer.valueOf()，把拆箱替换为 Integer.intValue()

```java
Integer integer = 1;
int unboxing = integer ++;
```

反编译输出：

```java

1: invokestatic  #2                  // Method
java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
8: invokevirtual #3                  // Method
java/lang/Integer.intValue:()I
```



Integer a = 1000; a++;几次拆箱和装箱

为什么要有Integer，那为什么又要有int

## Long 和 long的区别和 == 的比较 （是值比较）

和int类似

## final关键字修饰类，方法等的限制

## 不变类

   不要提供任何可以修改对象属性的方法  

   不要为属性提供set方法  

   保证类不会被扩展  

   用final修饰类  

   使所有的属性都是final的  

   初始化之后不能修改属性的值  

   使所有的属性都是private的  

   防止别的对象访问不可变类的属性  

   确保对任何可变组件的互斥访问  

   如果有指向可变对象的属性，一定要确保，这个可变对象不能被其他类访问和修改。最好不要在不可变类中添加 指向可变类的属性  

   优点 

   线程安全  

   不可变，线程间可共享，不存在多线程问题  

   易于设计，使用简单等等  

   没有多线程问题，不能被继承，不能被扩展，设计和使用的时候只需要针对当前需求就已经足够了。  

   缺点  

   每个不同的值都需要一个单独的对象

## java是单继承的还是多继承的，为什么是单继承的？

单继承。单继承结构中所有的对象都具有一个共用接口，所以归根到底都是相同的基本类型。

单继承结构使垃圾收集器的实现变得容易多了，由于所有的对象都保证具有其类型信息，因此不会因无法确定对象的类型而陷入僵局。

## lock，notify，等为什么要放在根类下面呢

## java8中接口中有默认实现的方法，为什么呢

主要是增加 Lambda、Stream 相关的功能

## java8新添了什么新的东西

## servlet你描述一下

## final ，finalize，finnaly的区别，是不是不可以改的，怎么可以改，

## 并发包类

## 你对java后续版本有什么建议？

## 

## 

## hashcode与equals方法的区别

## 注解的使用

## 基本数据类型

## 

## Java的aop的实现原理,两种代理机制的差别（实现原理）

## 

## JDK1.8新特性，主要问了流Stream

## maven的scope有几种

## OOA，OOD，OOP是什么说一下

## CAS比分段锁好在哪里，缺点又是什么

## 如何避免CAS一直自旋消耗资源

## 构造函数能不能继承

## 对象的比较是用的什么原理比较吗

## Java是值传递还是引用传递

## 深克隆和浅克隆

## 内部类实现单例，为什么用内部静态类

## 声明一个切面函数怎么声明

![1590139690044](java基础/1590139690044.png)

## 阿里有个证书，可以了解一下

## Java的缺点，跟c++和c比？

## final关键字

## 

## 一段代码,判断为什么会产生OOM异常

## 一段代码，判断三处可能异常的地方

## 写过注解没

## 反射的原理

## 什么是强引用？什么是弱引用？

## 笔试题里的ReentrantLock和Condition类似功能的还有啥机制

## 

## 水平触发和边缘触发的区别及各自的适用场景，

## 

## 

## 数组最大大小

## 一个`.java`文件从编译到执行的流程

## JVM内存模型，jdk1.7和1.8有什么区别

## jdk9或10有了解吗

## Rest成熟度模型

## Java 的值传递和引用传递



## Java线程间的通信

## Java .class文件结构

能问这个我是真没想到，还好之前看过的还没完全忘掉。推荐《深入理解JVM虚拟机》，class文件头两个字节是0xCAFEBABE固定的，然后是支持的最低虚拟机版本，然后是常量池，字符串之类的。。。然后是访问标志位，比如是否public是否static，然后是属性和方法的定义，不过当时这部分没看了，用不上。  

## 

## 能不能自己实现一个java.lang.String并加载

