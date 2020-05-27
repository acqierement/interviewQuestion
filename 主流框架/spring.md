# spring

## springboot和spring的区别

用springboot，和springmvc和spring有什么区别



   spring boot就是一个大框架里面包含了许许多多的东西，其中spring就是最核心的内容之一，当然就包含spring mvc。  

   spring mvc 是只是spring 处理web层请求的一个模块。  

   因此他们的关系大概就是这样：  

   spring mvc < spring <springboot。spring boot 我理解就是把 spring spring mvc spring data jpa 等等的一些常用的常用的基础框架组合起来，提供默认的配置，然后提供可插拔的设计，就是各种 starter ，来方便开发者使用这一系列的技术，套用官方的一句话， spring 家族发展到今天，已经很庞大了，作为一个开发者，如果想要使用 spring 家族一系列的技术，需要一个一个的搞配置，然后还有个版本兼容性问题，其实挺麻烦的，偶尔也会有小坑出现，其实蛮影响开发进度， spring boot 就是来解决这个问题，提供了一个解决方案吧，可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入 starter 就配置好了，直接使用，可以说追求开箱即用的效果吧.  

   spring 框架有超多的延伸产品例如 boot security jpa etc... 但它的基础就是 spring 的 ioc 和 aop ioc 提供了依赖注入的容器 aop 解决了面向横切面的编程 然后在此两者的基础上实现了其他延伸产品的高级功能 Spring MVC 呢是基于 Servlet 的一个 MVC 框架 主要解决 WEB 开发的问题 因为 Spring 的配置太复杂了 各种 XML JavaConfig hin 麻烦 于是懒人改变世界推出了 Spring boot 约定优于配置 简化了 spring 的配置流程.  

   Spring 最初利用“工厂模式”（ DI ）和“代理模式”（ AOP ）解耦应用组件。大家觉得挺好用，于是按照这种模式搞了一个 MVC 框架（一些用 Spring 解耦的组件），用开发 web 应用（ SpringMVC ）。然后有发现每次开发都要搞很多依赖，写很多样板代码很麻烦，于是搞了一些懒人整合包（ starter ），这套就是 Spring Boot 。

## spring的各种标签之间是什么，区别，bean和beanfactory的区别，继而引向工厂类，设计模式，解释一下怎么用的

传统的Spring做法是使用.xml文件来对bean进行注入或者是配置aop、事物，这么做有两个缺点：
1、如果所有的内容都配置在.xml文件中，那么.xml文件将会十分庞大；如果按需求分开.xml文件，那么.xml文件又会非常多。总之这将导致配置文件的可读性与可维护性变得很低。
2、在开发中在.java文件和.xml文件之间不断切换，是一件麻烦的事，同时这种思维上的不连贯也会降低开发的效率。
为了解决这两个问题，Spring引入了注解，通过"@XXX"的方式，让注解与Java Bean紧密结合，既大大减少了配置文件的体积，又增加了Java Bean的可读性与内聚性。
javabean：
3、JavaBean的含义
JavaBean是使用Java语言开发的一个可重用组件，能使Html代码与JAVA代码分离，并节省开发时间，简单的说就是一个包含了setter和getter以及至少一个无参构造方法的JAVA类，在框架中或其他方面也管它叫做PO，VO，TO等。
4、beanfactory：
BeanFactory是一个factory，spring简单工厂模式的接口类，spring IOC特性核心类，提供从工厂类中获取bean的各种方法，是所有bean的容器。它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。
FactoryBean： 是一个 Bean，实现了 FactoryBean 接口的类有能力改变 bean，
注解意思：
@Component(value="userDao")
public class UserDaoImpl implements UserDao {
@Override  public void sayHello() {
System.out.println("Hello Spring Annotation...");
}
}
@Component:组件.(作用在类上)
Spring 中提供@Component 的三个衍生注解:(功能目前来讲是一致的) *
@Controller :WEB 层 * @Service  :业务层 * @Repository :持久层
这三个注解是为了让标注类本身的用途清晰，Spring 在后续版本会对其增强

属性注入的注解:(使用注解注入的方式,可以不用提供 set 方法.)
@Value  :用    于注入普通类型. @Autowired :自动装配:
* 默认按类型进行装配.
*按名称注入:
* @Qualifier:强制使用名称注入.
@Resource 相当于: * @Autowired 和@Qualifier 一起使用

Bean 的作用范围的注解:
@Scope:
* singleton:单例 * prototype:多例

-》深入了解下设计模式，很重要！！！在接下来的时间里实践下设计模式，多多看下。

## spring几，和之前的什么区别

## 知道反射么，aop拦截点东西用反射怎么搞的。反射都可以得到哪些内容，链接java8新内容

## spring bean的生命周期

## springaop，怎么实现，写一个静态代理和动态代理的代码

## spring中bean的加载过程，springboot好处

## spring的循环依赖

## 事务传播级别

## 讲了一下Spring全家桶，比如AOP、IoC等特性和实现，还有Spring Boot的配置特性

## Spring如何使用事务？

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



Spring的事务是怎么管理的？

在方法上加注解

那为什么加注解可以实现，这个加载机制是怎么实现的，源码级别

为什么这个注解可以实现事务

那这个事务和数据库的事务有什么关联

## SpringBoot源码解读，启动方式，配置顺序等

## SpringMVC和Spring父子容器的关系

## 如何自定义实现SpringBoot中的starter

## JDK动态代理 与 CGLib动态代理

## 针对spring框架中bean的生命周期，如何不使用spring配置生命周期的功能，完成每个request与session都是单例的情况（利用反射生成匿名类）

## ASM怎么实现cglib

Spring的事务是怎么管理的

## spring的生命周期，不是bean

## SpringBoot，spring，springmvc三者关系

## 基于spring的应用，我想让这个程序启动之后，然后再打印一个log，或者一个定时任务，这个怎么实现

## AOP 两种***的区别，什么时候用

## SpringBoot注解，还问了底层实现原理

## 介绍MVC设计模式

##  IOC的流程

## AOP的失效问题？

classA里有个methodA和methodB，methodA调用了methodB，methodB有切面，ClassB去new一个ClassA  a,去调用a.methodA,那切面还会在吗？

## spring中哪里用到了工厂模式

## springboot的优点

## beanfactoryprostprocessor

## 关于spring问了对AplicationContextAware的了解

## SpringBoot特性，自动配置原理

## SpringBoot 和 SpringMVC 的区别

## SpringBoot 有深入了解吗？和 Spring Cloud 有什么差别吗？

## SpringBoot 核心框架包含什么？SpringCloud 一套微服务的框架中间有什么部分你是比较熟悉的，详细介绍一下。

## pringBoot 容器启动的大致流程（这个不会）

## SpringCloud 在 SpringBoot 的基础上扩展了什么？

（我提到了注册中心，作用是什么说了下）

## 分布式的Spring Cloud有了解过吗？

## spring mvc 和Servet的关系

## Spring 中的设计模式吗

（根据源码说了三种，单例，策略，工厂）

## Spring中加载流程

循环依赖怎么解决

## Spring的IOC，自己实现呢

## Spring的bean初始化流程，源码都有什么接口

## 三级缓存解决循环依赖

## Spring中Bean创建中可能出现的冲突问题Spring是如何解决的

