# spring

## spring的各种标签之间是什么，区别，bean和beanfactory的区别，继而引向工厂类，设计模式，解释一下怎么用的

### BeanFactory

BeanFactory，以Factory结尾，表示它是一个工厂类(接口)， **它负责生产和管理bean的一个工厂**。在Spring中，**BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等**，其中XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。XmlBeanFactory类将持有此XML配置元数据，并用它来构建一个完全可配置的系统或应用。   

### FactoryBean

**Spring提供了一个factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现**。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean\<T>的形式

以Bean结尾，表示它是一个Bean，不同于普通Bean的是：它是实现了FactoryBean\<T>接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个&符号来获取。

@Component:组件.(作用在类上)
Spring 中提供@Component 的三个衍生注解:(功能目前来讲是一致的) 
@Controller :WEB 层   @Service  :业务层   @Repository :持久层
这三个注解是为了让标注类本身的用途清晰，Spring 在后续版本会对其增强

属性注入的注解:(使用注解注入的方式,可以不用提供 set 方法.)
@Value  :用于注入普通类型. @Autowired :自动装配:

* 默认按类型进行装配.
按名称注入:
* @Qualifier:强制使用名称注入.
@Resource 相当于: * @Autowired 和@Qualifier 一起使用

Bean 的作用范围的注解:
@Scope: singleton:单例  prototype:多例

-》深入了解下设计模式，很重要！！！在接下来的时间里实践下设计模式，多多看下。

## spring几，和之前的什么区别

Spring 2.x增加对注解的支持，支持了基于注解的配置。

Spring 3.x支持了基于Java类的配置。

Spring 4.x全面支持Java 8.0，支持Lambda表达式的使用，提供了对@Scheduled和@PropertySource重复注解的支持，提供了空指针终结者Optional，对核心容器进行增加：支持泛型的依赖注入、Map的依赖注入、Lazy延迟依赖的注入、List注入、Condition条件注解注入、对CGLib动态代理类进行了增强。

Spring 4.x开始，Spring MVC基于Servlet 3.0 开发，并且为了方便Restful开发，引入了新的RestController注解器注解，同时还增加了一个AsyncRestTemplate支持Rest客户端的异步无阻塞请求。

Spring5 的基准版本为8。支持响应式编程支持。增加函数式web框架，该框架引入了两个基本组件：HandlerFunction 和 RouterFunction。

## @RestController注解

@RestController注解，相当于@Controller+@ResponseBody两个注解的结合，返回json数据不需要在方法前面加@ResponseBody注解了，但使用@RestController这个注解，就不能返回jsp,html页面，视图解析器无法解析jsp,html页面

## **@Scope有几种模式**

**1.singleton单例模式,**

　　全局有且仅有一个实例

**2.prototype原型模式，**

　　每次获取Bean的时候会有一个新的实例

**3.request**　　

​    request表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效，

**4.session**　

​     session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效

**5.global session**

​     global session作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个 portlet web应用的各种不同的portlet所共享。在global session作用域中定义的bean被限定于全局portlet Session的生命周期范围内。如果你在web中使用global session作用域来标识bean，那么web会自动当成session类型来使用。

## spring中bean的加载过程

> 具体可以看本目录下的`spring中bean的加载过程`

### Spring的bean初始化流程，源码都有什么接口

## spring bean的生命周期

> 具体可以看本目录下的`spring bean的生命周期`

## spring的循环依赖

> 具体可以看本目录下的spring的循环依赖

对于构造器注入，在getSingleton()方法中会调用beforeSingletonCreation()方法去做检查，在这里出现异常报错。

```java
if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
	throw new BeanCurrentlyInCreationException(beanName);
}

// singletonsCurrentlyInCreation的结构是这样的。
/** Names of beans that are currently in creation */
private final Set<String> singletonsCurrentlyInCreation =      Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```

可以看到每次getSingleton的时候，会先在singletonsCurrentlyInCreation加入当前bean，如果之前已经存在的话，就会报错。

```java
/** Names of beans that are currently in creation */
private final ThreadLocal<Object> prototypesCurrentlyInCreation = new NamedThreadLocal<>("Prototype beans currently in creation");
```

## aop

### 知道反射么，aop拦截点东西用反射怎么搞的。反射都可以得到哪些内容，链接java8新内容

通过java 反射技术，可以获取很多信息：

1.类名

2.属性，属性名

3.方法，方法名

4.注解

### springaop，怎么实现，写一个静态代理和动态代理的代码

### AOP的失效问题？

classA里有个methodA和methodB，methodA调用了methodB，methodB有切面，ClassB去new一个ClassA  a,去调用a.methodA,那切面还会在吗？

因为不是通过代理类去调用，所以aop失效

### 事务的传播机制？

PROPAGATION_REQUIRED --支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。

PROPAGATION_SUPPORTS--支持当前事务，如果当前没有事务，就以非事务方式执行。

PROPAGATION_MANDATORY--支持当前事务，如果当前没有事务，就抛出异常。

PROPAGATION_REQUIRES_NEW--新建事务，如果当前存在事务，把当前事务挂起。

PROPAGATION_NOT_SUPPORTED--以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 

PROPAGATION_NEVER--以非事务方式执行，如果当前存在事务，则抛出异常。

PROPAGATION_NESTED -- 理解Nested的关键是savepoint。他与PROPAGATION_REQUIRES_NEW的区别是，PROPAGATION_REQUIRES_NEW另起一个事务，将会与他的父事务相互独立，而Nested的事务和他的父事务是相依的，他的提交是要等和他的父事务一块提交的。也就是说，如果父事务最后回滚，他也要回滚的。
而Nested事务的好处是他有一个savepoint。

### Spring如何使用事务？Spring的事务是怎么管理的

#### Spring的事务实现方式

使用`@Transactional`

那为什么加注解可以实现，这个加载机制是怎么实现的，源码级别

为什么这个注解可以实现事务

那这个事务和数据库的事务有什么关联

#### 讲了一下`@Transactional`的使用方法

| 属性名           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| value            | 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。 |
| propagation      | 事务的传播行为，默认值为 REQUIRED。                          |
| isolation        | 事务的隔离度，默认值采用 DEFAULT。                           |
| timeout          | 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| read-only        | 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。 |
| rollback-for     | 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。 |
| no-rollback- for | 抛出 no-rollback-for 指定的异常类型，不回滚事务。            |

#### @Transactional实现原理

https://blog.csdn.net/qq_20597727/article/details/84868035

@Transactional的实现是基于aop的。spring会生成一个**BeanFactoryTransactionAttributeSourceAdvisor实例**，这个实例可以看做一个切点。

aop会在`AopUtils#findAdvisorsThatCanApply`中判断切面是否适用当前bean。如果有@Transactional的注解就表示适用这个advisor，然后会开启一个事务来执行方法。等同于一个@Around

#### spring事务传播行为怎么实现

#### 事务的注意事项？

`public`方法、自调用无效、回滚一致性、异常处理机制

**public方法**

Spring会检查目标方法的修饰符是不是 public，若不是 public，就不会获取@Transactional 的属性配置信息，最终会造成不会用 TransactionInterceptor 来拦截该目标方法进行事务管理。见下面代码：

```
protected TransactionAttribute computeTransactionAttribute(Method method,
    Class<?> targetClass) {
        // Don't allow no-public methods as required.
        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
return null;}
```

**自调用无效**

在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有@Transactional 注解的方法内部调用有@Transactional 注解的方法，有@Transactional 注解的方法的事务被忽略，不会发生回滚。

为解决public和自调用无效这两个问题，可以使用 AspectJ 取代 Spring AOP 代理。需要将 AspectJ 信息添加到 xml 配置信息中。

#### 什么时候会触发异常回滚？

默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。

如果在事务中抛出其他类型的异常，并期望 Spring 能够回滚事务，可以指定 rollbackFor。

通过分析 Spring 源码可以知道，若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚。

#### 如何实现分布式事务？

TCC分布式事务，try、commit、cancel，利用补偿机制和幂等性解决事务一致性问题

#### Spring的事务是怎么管理的？



## 讲了一下Spring全家桶，比如AOP、IoC等特性和实现

## SpringMVC和Spring父子容器的关系

[spring与springmvc父子容器](http://www.tianshouzhi.com/api/tutorials/spring)

## Springmvc在哪里处理异常

## 发出一个url请求服务器怎么接收和处理

http请求到达api的映射问题

## JDK动态代理 与 CGLib动态代理

## spring的生命周期，不是bean

> 具体可以看本目录下的spring启动流程

## 基于spring的应用，我想让这个程序启动之后，然后再打印一个log，或者一个定时任务，这个怎么实现

实现ApplicationListener

```java
@Component
public class ApplicationStartQuartzJobListener implements ApplicationListener<ContextRefreshedEvent>{
 
	@Autowired
    private QuartzManager quartzManager;
 
    /**
     * 初始启动quartz
     */
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        try {
        	quartzManager.start();
            System.out.println("任务已经启动...");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

springboot的话还可以实现CommandLineRunner，ApplicationRunner 

**CommandLineRunner**

```java
@Component
public class StartPingService implements CommandLineRunner{
 
	@Autowired
	Ping ping;
	
	@Override
	public void run(String... args) throws Exception {
		// TODO Auto-generated method stub
		ping.pingStart();
	}
 
}
```

**ApplicationRunner** 

```java
@Component
public class JDDRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(args);
        System.out.println("这个是测试ApplicationRunner接口");
    }
}
```

## 介绍MVC设计模式

##  IOC的流程

自己实现呢？

## BeanFactoryPostProcessor

里面定义了一个接口

```java
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
```

可以对beanFactory的数据进行修改

## 对ApplicationContextAware的了解

Aware就是意识到，感知到的意思。ApplicationContextAware表示实现这个接口就可以得到ApplicationContext。

那什么时候去调用这个方法呢？

在初始化的时候，也就是在initializeBean()方法中会调用invokeAwareMethods(beanName, bean)方法，去调用各个aware的方法。

## spring mvc 和servlet的关系

## Spring 中的设计模式

### spring中哪里用到了工厂模式

（根据源码说了三种，单例，策略，工厂）

## 针对spring框架中bean的生命周期，如何不使用spring配置生命周期的功能，完成每个request与session都是单例的情况（利用反射生成匿名类）

不知道

## ASM怎么实现cglib

不知道

# SpringBoot

## Spring Boot的配置特性

## SpringBoot，spring，springmvc三者关系

### SpringBoot 和 SpringMVC 的区别

### SpringBoot 和 spring的区别

用springboot，和springmvc和spring有什么区别

   spring boot就是一个大框架里面包含了许许多多的东西，其中spring就是最核心的内容之一，当然就包含spring mvc。  

   spring mvc 是只是spring 处理web层请求的一个模块。  

   因此他们的关系大概就是这样：  

   spring mvc < spring <springboot。spring boot 我理解就是把 spring spring mvc spring data jpa 等等的一些常用的常用的基础框架组合起来，提供默认的配置，然后提供可插拔的设计，就是各种 starter ，来方便开发者使用这一系列的技术，套用官方的一句话， spring 家族发展到今天，已经很庞大了，作为一个开发者，如果想要使用 spring 家族一系列的技术，需要一个一个的搞配置，然后还有个版本兼容性问题，其实挺麻烦的，偶尔也会有小坑出现，其实蛮影响开发进度， spring boot 就是来解决这个问题，提供了一个解决方案吧，可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入 starter 就配置好了，直接使用，可以说追求开箱即用的效果吧.  

   spring 框架有超多的延伸产品例如 boot security jpa etc... 但它的基础就是 spring 的 ioc 和 aop ioc 提供了依赖注入的容器 aop 解决了面向横切面的编程 然后在此两者的基础上实现了其他延伸产品的高级功能 Spring MVC 呢是基于 Servlet 的一个 MVC 框架 主要解决 WEB 开发的问题 因为 Spring 的配置太复杂了 各种 XML JavaConfig hin 麻烦 于是懒人改变世界推出了 Spring boot 约定优于配置 简化了 spring 的配置流程.  

   Spring 最初利用“工厂模式”（ DI ）和“代理模式”（ AOP ）解耦应用组件。大家觉得挺好用，于是按照这种模式搞了一个 MVC 框架（一些用 Spring 解耦的组件），用开发 web 应用（ SpringMVC ）。然后有发现每次开发都要搞很多依赖，写很多样板代码很麻烦，于是搞了一些懒人整合包（ starter ），这套就是 Spring Boot 。

## SpringBoot注解，还问了底层实现原理

## SpringBoot源码解读，启动方式，配置顺序等

### SpringBoot 容器启动的大致流程

## SpringBoot特性，自动配置原理

## springboot的优点

Spring Boot 的最大的优势是“约定优于配置“。“约定优于配置“是一种软件设计范式，开发人员按照约定的方式来进行编程，可以减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。

内嵌tomcat，不需要部署war文件

提供start来简化搭建配置

不需要代码生成，也不需要xml配置

## 如何自定义实现SpringBoot中的starter

建一个starter工程

定义一个配置类

```java
package com.spring.study;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.stereotype.Component;

@Component
@ComponentScan("com.spring.study.module")
public class HelloServiceAutoConfiguration {
}
```

在META-INF/spring.factories中配置这个类

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.spring.study.HelloServiceAutoConfiguration
```

然后引入这个工程就可以使用工程中定义的bean了

## SpringBoot 核心框架包含什么？

# SpringCloud

## SpringCloud 在 SpringBoot 的基础上扩展了什么？

（我提到了注册中心，作用是什么说了下）

## 分布式的Spring Cloud有了解过吗？

## 

## SpringBoot 有深入了解吗？和 Spring Cloud 有什么差别吗？

## SpringCloud 一套微服务的框架中间有什么部分你是比较熟悉的，详细介绍一下。

## Spring MVC的工作流程

Spring会默认为我们注册`RequestMappingHandlerMapping`等Bean定义。而`RequestMappingHandlerMapping`实现了`InitializingBean`接口,会执行他的afterPropertySet方法.

```java
	public void afterPropertiesSet() {  
        initHandlerMethods();  
    }  
  
    //Scan beans in the ApplicationContext, detect and register handler methods.  
    protected void initHandlerMethods() {  
        //扫描所有注册的Bean  
        String[] beanNames = (this.detectHandlerMethodsInAncestorContexts ?  
           BeanFactoryUtils.beanNamesForTypeIncludingAncestors(getApplicationContext(),   
                Object.class) : getApplicationContext().getBeanNamesForType(Object.class));  
        //遍历这些Bean，依次判断是否是处理器，并检测其HandlerMethod  
        for (String beanName : beanNames) {  
            if (isHandler(getApplicationContext().getType(beanName))){  
                detectHandlerMethods(beanName);  
            }  
        }  
        //这个方法是个空实现,不管他  
        handlerMethodsInitialized(getHandlerMethods());  
    }  
```

 整个的检测过程大致这样：1）遍历Handler中的所有方法，找出其中被@RequestMapping注解标记的方法。2）然后遍历这些方法，生成RequestMappingInfo实例。3）将RequestMappingInfo实例以及处理器方法注册到缓存中。

![img](spring面试题/1121080-20190509202147059-745656946.jpg)