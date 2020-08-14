

# spring

## spring的各种标签之间是什么，区别，bean和beanfactory的区别，继而引向工厂类，设计模式，解释一下怎么用的

### BeanFactory

BeanFactory，以Factory结尾，表示它是一个工厂类(接口)， **它负责生产和管理bean的一个工厂**。在Spring中，**BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如 DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，其中****XmlBeanFactory就是常用的一个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系**。XmlBeanFactory类将持有此XML配置元数据，并用它来构建一个完全可配置的系统或应用。   

### FactoryBean

**Spring提供了一个factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占用重要的地位，Spring自身就提供了70多个FactoryBean的实现**。它们隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始支持泛型，即接口声明改为FactoryBean<T>的形式

以Bean结尾，表示它是一个Bean，不同于普通Bean的是：它是实现了FactoryBean<T>接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加一个&符号来获取。

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

## spring中bean的加载过程

> 具体可以看本目录下的`spring中bean的加载过程`

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

### springaop，怎么实现，写一个静态代理和动态代理的代码

### AOP的失效问题？

classA里有个methodA和methodB，methodA调用了methodB，methodB有切面，ClassB去new一个ClassA  a,去调用a.methodA,那切面还会在吗？

## 事务

### 事务传播级别

### Spring如何使用事务？Spring的事务是怎么管理的

Spring的事务实现方式

讲了一下`@Transactional`的使用方法和实现原理

事务的注意事项？
`public`方法、自调用无效、回滚一致性、异常处理机制

事务的传播机制？
七种传播机制

什么时候会触发异常回滚？
事务方法抛出`RuntimeException`

如何实现分布式事务？
TCC分布式事务，try、commit、cancel，利用补偿机制和幂等性解决事务一致性问题

#### Spring的事务是怎么管理的？

在方法上加注解

那为什么加注解可以实现，这个加载机制是怎么实现的，源码级别

为什么这个注解可以实现事务

那这个事务和数据库的事务有什么关联

## 讲了一下Spring全家桶，比如AOP、IoC等特性和实现

## SpringMVC和Spring父子容器的关系

[spring与springmvc父子容器](http://www.tianshouzhi.com/api/tutorials/spring)

## JDK动态代理 与 CGLib动态代理

## 针对spring框架中bean的生命周期，如何不使用spring配置生命周期的功能，完成每个request与session都是单例的情况（利用反射生成匿名类）

不知道

## ASM怎么实现cglib

## spring的生命周期，不是bean

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

## spring中哪里用到了工厂模式

## beanfactorypostprocessor

## 对ApplicationContextAware的了解

## spring mvc 和Servet的关系

## Spring 中的设计模式吗

（根据源码说了三种，单例，策略，工厂）

## Spring的IOC，自己实现呢

## Spring的bean初始化流程，源码都有什么接口

## Spring中Bean创建中可能出现的冲突问题Spring是如何解决的

# SpringBoot

## springboot和spring的区别

用springboot，和springmvc和spring有什么区别

   spring boot就是一个大框架里面包含了许许多多的东西，其中spring就是最核心的内容之一，当然就包含spring mvc。  

   spring mvc 是只是spring 处理web层请求的一个模块。  

   因此他们的关系大概就是这样：  

   spring mvc < spring <springboot。spring boot 我理解就是把 spring spring mvc spring data jpa 等等的一些常用的常用的基础框架组合起来，提供默认的配置，然后提供可插拔的设计，就是各种 starter ，来方便开发者使用这一系列的技术，套用官方的一句话， spring 家族发展到今天，已经很庞大了，作为一个开发者，如果想要使用 spring 家族一系列的技术，需要一个一个的搞配置，然后还有个版本兼容性问题，其实挺麻烦的，偶尔也会有小坑出现，其实蛮影响开发进度， spring boot 就是来解决这个问题，提供了一个解决方案吧，可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入 starter 就配置好了，直接使用，可以说追求开箱即用的效果吧.  

   spring 框架有超多的延伸产品例如 boot security jpa etc... 但它的基础就是 spring 的 ioc 和 aop ioc 提供了依赖注入的容器 aop 解决了面向横切面的编程 然后在此两者的基础上实现了其他延伸产品的高级功能 Spring MVC 呢是基于 Servlet 的一个 MVC 框架 主要解决 WEB 开发的问题 因为 Spring 的配置太复杂了 各种 XML JavaConfig hin 麻烦 于是懒人改变世界推出了 Spring boot 约定优于配置 简化了 spring 的配置流程.  

   Spring 最初利用“工厂模式”（ DI ）和“代理模式”（ AOP ）解耦应用组件。大家觉得挺好用，于是按照这种模式搞了一个 MVC 框架（一些用 Spring 解耦的组件），用开发 web 应用（ SpringMVC ）。然后有发现每次开发都要搞很多依赖，写很多样板代码很麻烦，于是搞了一些懒人整合包（ starter ），这套就是 Spring Boot 。

## 还有Spring Boot的配置特性



## SpringBoot，spring，springmvc三者关系

## springboot好处

## SpringBoot注解，还问了底层实现原理

## SpringBoot源码解读，启动方式，配置顺序等

## SpringBoot特性，自动配置原理

## springboot的优点

## 如何自定义实现SpringBoot中的starter

## SpringBoot 和 SpringMVC 的区别

## SpringBoot 有深入了解吗？和 Spring Cloud 有什么差别吗？

## SpringBoot 核心框架包含什么？SpringCloud 一套微服务的框架中间有什么部分你是比较熟悉的，详细介绍一下。

## SpringBoot 容器启动的大致流程（这个不会）

# SpringCloud

## SpringCloud 在 SpringBoot 的基础上扩展了什么？

（我提到了注册中心，作用是什么说了下）

## 分布式的Spring Cloud有了解过吗？

## 

