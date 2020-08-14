# spring启动流程

参考[spring容器的refresh方法分析](https://www.cnblogs.com/grasp/p/11942735.html)

```java
synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing, setting its startup date and active flag as well as performing any initialization of property sources.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
     		}
```

## prepareRefresh方法

设置开始时间和启动标志，执行属性的初始化

## obtainFreshBeanFactory方法

刷新BeanFactory，并获取Spring上下文的Bean工厂

## prepareBeanFactory方法

给该工厂配置一些基本的特性，比如设置一个类加载器和后置处理器

## postProcessBeanFactory方法

上面对bean工厂进行了许多配置，现在需要对bean工厂进行一些处理。不同的Spring容器做不同的操作。比如GenericWebApplicationContext容器的操作会在BeanFactory中添加ServletContextAwareProcessor用于处理ServletContextAware类型的bean初始化的时候调用setServletContext或者setServletConfig方法(跟ApplicationContextAwareProcessor原理一样)。

## invokeBeanFactoryPostProcessors方法

在Spring容器中找出实现了BeanFactoryPostProcessor接口的processor并执行

> 注
> 1.在springboot的web程序初始化AnnotationConfigServletWebServerApplicationContext容器时，会初始化内部属性AnnotatedBeanDefinitionReader reader，这个reader构造的时候会在BeanFactory中注册一些post processor，包括BeanPostProcessor和BeanFactoryPostProcessor(比如ConfigurationClassPostProcessor、AutowiredAnnotationBeanPostProcessor)：
>
> ```java
> AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
> ```
>
> 在使用mybatis时，一般配置了MapperScannerConfigurer的bean，这个bean就是继承的BeanDefinitionRegistryPostProcessor，所以也是这个地方把扫描的mybatis的接口注册到容器中的。

invokeBeanFactoryPostProcessors方法处理BeanFactoryPostProcessor的逻辑如下：

从Spring容器中找出BeanDefinitionRegistryPostProcessor类型的bean(这些processor是在容器刚创建的时候通过构造AnnotatedBeanDefinitionReader的时候注册到容器中的)，然后按照优先级分别执行，优先级的逻辑如下：

1. 实现PriorityOrdered接口的BeanDefinitionRegistryPostProcessor先全部找出来，然后排序后依次执行
2. 实现Ordered接口的BeanDefinitionRegistryPostProcessor找出来，然后排序后依次执行
3. 没有实现PriorityOrdered和Ordered接口的BeanDefinitionRegistryPostProcessor找出来执行并依次执行

## registerBeanPostProcessors方法

   从Spring容器中找出实现BeanPostProcessor接口的bean，并设置到BeanFactory的属性中。之后bean被实例化的时候会调用这个BeanPostProcessor。

## initMessageSource方法

初始化MessageSource组件（做国际化功能；消息绑定，消息解析）,这个接口提供了消息处理功能。主要用于国际化/i18n。

## initApplicationEventMulticaster方法

在Spring容器中初始化事件广播器，事件广播器用于事件的发布。

程序首先会检查bean工厂中是否有bean的名字和这个常量(applicationEventMulticaster)相同的，如果没有则说明没有那么就使用默认的ApplicationEventMulticaster 的实现：SimpleApplicationEventMulticaster

## onRefresh方法

一个模板方法，不同的Spring容器做不同的事情。

比如web程序的容器ServletWebServerApplicationContext中会调用createWebServer方法去创建内置的Servlet容器。

目前SpringBoot只支持3种内置的Servlet容器：

- Tomcat
- Jetty
- Undertow

## registerListeners方法

注册应用的监听器。就是注册实现了ApplicationListener接口的监听器bean，这些监听器是注册到ApplicationEventMulticaster中的。这不会影响到其它监听器bean。在注册完以后，还会将其前期的事件发布给相匹配的监听器。

## finishBeanFactoryInitialization方法

实例化BeanFactory中已经被注册但是未实例化的所有实例(懒加载的不需要实例化)。

比如invokeBeanFactoryPostProcessors方法中根据各种注解解析出来的类，在这个时候都会被初始化。

实例化的过程各种BeanPostProcessor开始起作用。

## finishRefresh方法

refresh做完之后需要做的其他事情。

- 初始化生命周期处理器，并设置到Spring容器中(LifecycleProcessor)
- 调用生命周期处理器的onRefresh方法，这个方法会找出Spring容器中实现了SmartLifecycle接口的类并进行start方法的调用
- 发布ContextRefreshedEvent事件告知对应的ApplicationListener进行响应的操作

如果是web容器ServletWebServerApplicationContext还会启动web服务和发布消息