# Spring Aop实现原理

通过对配置文件的解析<aop:aspectj-autoproxy /> ，会去注册AnnotationAwareAspectJAutoProxyCreator类的自动注册。

AnnotationAwareAspectJAutoProxyCreator实现了BeanPostProcessor接口，在初始化后会调用postProcessAfterInitialization。

父类AbstractAutoProxyCreator的postProcessAfterInitialization方法：

```java
	@Override
	public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (this.earlyProxyReferences.remove(cacheKey) != bean) {
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}
```

主要是在getAdvicesAndAdvisorsForBean方法中获取代理。

getAdvicesAndAdvisorsForBean方法中的findEligibleAdvisors方法。

要获取指定bean的增强方法，需要两步，获取所有的增强，以及获取适用于当前bean的增强并应用。就是下面的两个方法：

```java
List<Advisor> candidateAdvisors = findCandidateAdvisors();
List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
```

先看看AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors

```java
protected List<Advisor> findCandidateAdvisors() {
    // Add all the Spring advisors found according to superclass rules.
    // 当使用注释方式配置 AOP 的时候并不是丢弃了对 XML 配置的支持。
    // 在这里调用父类方法加载配置文件中的 AOP 声明
    List<Advisor> advisors = super.findCandidateAdvisors();

    // Build Advisors for all AspectJ aspects in the bean factory.
    // 把BeanFactory里面的所有定义切面，定义成Spring中用的切面（Advisor），并放到集合里
    advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
    return advisors;
}

	/**
	 * Look for AspectJ-annotated aspect beans in the current bean factory,
	 * and return to a list of Spring AOP Advisors representing them.
	 * <p>Creates a Spring Advisor for each AspectJ advice method.
	 * @return the list of {@link org.springframework.aop.Advisor} beans
	 * @see #isEligibleBean
	 */
	public List<Advisor> buildAspectJAdvisors() {
		List<String> aspectNames = this.aspectBeanNames;

		if (aspectNames == null) {
			synchronized (this) {
				aspectNames = this.aspectBeanNames;
				if (aspectNames == null) {
					List<Advisor> advisors = new ArrayList<>();
					aspectNames = new ArrayList<>();
            // beanNamesForTypeIncludingAncestors方法是通过循环所有的bean，
            // 来取得所有指定类型的子类（这里指定的类型是Object，所以是所有类）
            // 实现是DefaultListableBeanFactory#doGetBeanNamesForType方法中
            // 判断子类是使用的Class#isAssignableFrom方法
					String[] beanNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
							this.beanFactory, Object.class, true, false);
					for (String beanName : beanNames) {
						if (!isEligibleBean(beanName)) {
							continue;
						}
						// We must be careful not to instantiate beans eagerly as in this case they
						// would be cached by the Spring container but would not have been weaved.
                        // 获取对应的 bean 的类型
						Class<?> beanType = this.beanFactory.getType(beanName);
						if (beanType == null) {
							continue;
						}
                         // 如果beanType是Aspect类型的话
						if (this.advisorFactory.isAspect(beanType)) {
							aspectNames.add(beanName);
							AspectMetadata amd = new AspectMetadata(beanType, beanName);
							if (amd.getAjType().getPerClause().getKind() == PerClauseKind.SINGLETON) {
								MetadataAwareAspectInstanceFactory factory =
										new BeanFactoryAspectInstanceFactory(this.beanFactory, beanName);
                                // 读取Aspect类的方法上的注解，从而生成Advisors。
								List<Advisor> classAdvisors = this.advisorFactory.getAdvisors(factory);
								if (this.beanFactory.isSingleton(beanName)) {
									this.advisorsCache.put(beanName, classAdvisors);
								}
								else {
									this.aspectFactoryCache.put(beanName, factory);
								}
								advisors.addAll(classAdvisors);
							}
							else {
								// Per target or per this.
								if (this.beanFactory.isSingleton(beanName)) {
									throw new IllegalArgumentException("Bean with name '" + beanName +
											"' is a singleton, but aspect instantiation model is not singleton");
								}
								MetadataAwareAspectInstanceFactory factory =
										new PrototypeAspectInstanceFactory(this.beanFactory, beanName);
								this.aspectFactoryCache.put(beanName, factory);
								advisors.addAll(this.advisorFactory.getAdvisors(factory));
							}
						}
					}
                    // 将找到的beanName缓存起来
					this.aspectBeanNames = aspectNames;
					return advisors;
				}
			}
		}

		if (aspectNames.isEmpty()) {
			return Collections.emptyList();
		}
		List<Advisor> advisors = new ArrayList<>();
		for (String aspectName : aspectNames) {
			List<Advisor> cachedAdvisors = this.advisorsCache.get(aspectName);
			if (cachedAdvisors != null) {
				advisors.addAll(cachedAdvisors);
			}
			else {
				MetadataAwareAspectInstanceFactory factory = this.aspectFactoryCache.get(aspectName);
				advisors.addAll(this.advisorFactory.getAdvisors(factory));
			}
		}
		return advisors;
	}
```

findCandidateAdvisors方法中做了下面这些事情：

1. 找出所有的beanName（找一次就够了，会缓存起来）
2. 根据beanName获取所有的类型，找出所有声明Aspect的bean
3. 对标记为Aspect的类进行增强的处理
4. 将提取结果加入缓存

第三步对Aspect的类进行解析比较复杂，主要在getAdvisors方法中。

首先获取切点信息，就是@Before这些注解，然后根据切点信息生成增强器

spring会根据不同的注解生成不同的增强器，所有的增强器都是通过InstantiationModelAwarePointcutAdvisorImpl封装的，该类实现了Advisor这个接口，最后返回的就是Advisor类

## 寻找匹配的增强器

主要方法是在findAdvisorsThatCanApply中。根据advisor中的MethodMatcher来判断是否匹配。

## 创建代理

createProxy方法里面给ProxyFactory设置相应的属性，包括是否使用继承还是接口，后面判断是jdk代理还是动态代理

proxyFactory.getProxy判断是采用cglib还是jdk代理

```java
if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
进到这里就采用cglib
}
```

如果有optimize配置，或者有设置proxyTargetClass，以及是否存在代理接口

如果有接口，默认会采用jdk代理，当然也可以设置proxyTargetClass为true强制使用cglib。

### jdk代理

看一下jdk代理实现，主要方法在invoke方法里面

```java
// Get the interception chain for this method.
List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
MethodInvocation invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
// Proceed to the joinpoint through the interceptor chain.
retVal = invocation.proceed();
```

主要工作是创建了一个拦截器，通过ReflectiveMethodInvocation进行封装，然后再调用process去实现拦截器的逐一调用。这里主要就是职责链模式

