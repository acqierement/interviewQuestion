# java类加载顺序，一题全搞懂

来直接做题，看看会打印什么。大家自己把答案写下来，注意看清楚上下文，有些变量在最后面。

## 类结构

参考java编程思想5.7.2 静态数据的初始化，比原来的结构复杂一点。

**父类Parent**

```java
public class Parent {
    static Bowl bow5 = new Bowl(5);
    Bowl bow6 = new Bowl(6);
    static {
        System.out.println("执行Parent静态代码块");
    }

    {
        System.out.println("执行Parent构造代码块");
    }

    Parent() {
        System.out.println("执行Parent构造函数");
    }
}
```

**子类Cupboard**

```java
public class Cupboard extends Parent{
    Bowl bowl1 = new Bowl(1);
    static Cupboard cupboard = new Cupboard();
    static Bowl bowl2 = new Bowl(2);

    static {
        System.out.println("执行Cupboard静态代码块");
    }
    {
        System.out.println("执行Cupboard构造代码块");
    }
    public Cupboard() {
        System.out.println("Cupboard()");
        System.out.println("a=" + a + ",b=" + b);
    }

    public void f2(int mark) {
        System.out.println("f2(" + mark + ")");
    }

    static Bowl bowl3 = new Bowl(3);
    Bowl bowl4 = new Bowl(4);
    int a = 110;
    static int b = 200;
}
```

**辅助类Bowl（不重要）**

```java
public class Bowl {
    Bowl(int mark) {
        System.out.println(mark);
    }
}
```

## 答案

```
5
执行Parent静态代码块
6
执行Parent构造代码块
执行Parent构造函数
1
执行Cupboard构造代码块
4
Cupboard()
a=110,b=0
2
执行Cupboard静态代码块
3
6
执行Parent构造代码块
执行Parent构造函数
1
执行Cupboard构造代码块
4
Cupboard()
a=110,b=200
---------mian函数开始----------
6
执行Parent构造代码块
执行Parent构造函数
1
执行Cupboard构造代码块
4
Cupboard()
a=110,b=200
f2(1)
```

不知道你做对了没有？如果做对了，就不用看后面的讲解了。

下面我们来看看执行过程：

## 预备知识

首先我们要先知道类的加载过程，其中主要有影响的是准备阶段和初始化阶段。

准备阶段是为静态变量分配内存并赋初始值，初始值通常情况下是数据的零值。注意这时候进行内存分配只包括类变量，不包括实例变量。

初始化阶段是会完成所有类变量的赋值和执行静态代码块中的语句。准备阶段保证在初始化阶段之前。

## 主类

首先我们执行main函数，main函数所在的类作为启动的主类，会先被初始化，所以首先会执行静态变量的初始化。

```java
    static Cupboard cupboard = new Cupboard();
```

在new之前，因为这个类没有被初始化过，所以会先完成Cupboard类的初始化。

初始化Cupboard类的时候，发现父类没有初始化，就会先执行父类Parent的初始化。

## Parent类的初始化

类的初始化会按顺序执行静态变量和静态代码块。所以会按顺序执行：

```java
    static Bowl bow5 = new Bowl(5);
    static {
        System.out.println("执行Parent静态代码块");
    }
```

Bowl比较简单，在我们例子里的作用主要是为了打印，所以这里就不讨论这个类了。

到这里，Parent类的初始化就完成了。然后回到子类Cupboard的初始化，这里是重头戏。

## Cupboard类的初始化

Cupboard类的初始化还是执行静态变量和静态代码块。第一个就遇到了：

```java
    static Cupboard cupboard = new Cupboard();
```

在类的初始化过程中，遇到了实例化。这里实例化就不会再去加载Cupboard类了，因为只需要加载一次就够了。

所以接着就去执行类的实例化了。

### Cupboard实例化

实例化首先会按顺序执行实例变量的赋值和构造代码块的语句，构造函数的执行会在最后面。

在实例化子类的时候，会先完成父类Parent的实例化。

Parent的实例化就是按顺序完成实例变量的赋值和构造函数块，最后执行构造函数。

然后就是子类Cupboard的实例化，先完成实例变量bowl1和bowl4、a的赋值，打印1，4,并且a也有值了，再执行构造函数块，执行构造函数。这里构造函数中a和b的值是多少呢？

已经说了这里a已经有值了。但是因为Cupboard类的初始化还没有完成（还记得吗？在初始化第一步就遇到了“new Cupboard“）初始化开始前，已经确保执行了准备阶段，这时候b就有初始的0值，所以b的值这里就是0。

然后Cupboard的初始化就完成了。

### 继续初始化

这样，就可以回到最开始Cupboard的初始化。

把静态变量和静态代码块拿出来：

```java
    static Cupboard cupboard = new Cupboard();
    static Bowl bowl2 = new Bowl(2);
    static {
        System.out.println("执行Cupboard静态代码块");
    }
    static Bowl bowl3 = new Bowl(3);
```

我们前面说了那么多，只完成了第一步，然后再按顺序执行其他的语句。这样就完成了Cupboard类的初始化。

还记得为什么要初始化Cupboard类吗？main函数里面有这么一个类变量，因为这个类没有被加载过，所以才实例化之前要先初始化。

```
static Cupboard cupboard = new Cupboard();
```

然后才是执行new Cupboard()的实例化操作：

执行实例化其实我们前面已经说过了，再复习一遍。

## Cupboard实例化

首先实例化父类Parent。实例化先按顺序执行实例变量和构造代码块，最后执行构造函数。打印：

```
6
执行Parent构造代码块
执行Parent构造函数
```

然后实例化Cupboard，同样的，按顺序执行实例变量和构造代码块，最后执行构造函数。打印

```
1
执行Cupboard构造代码块
4
Cupboard()
a=110,b=200
```

## Main函数

然后才是执行main函数：

```java
    public static void main(String[] args) {
        new Cupboard();
        cupboard.f2(1);
    }
```

new Cupboard()实例化已经说过了，实例化会打印：

```
6
执行Parent构造代码块
执行Parent构造函数
1
执行Cupboard构造代码块
4
Cupboard()
a=110,b=200
```

最后执行函数，打印f2(1)。

