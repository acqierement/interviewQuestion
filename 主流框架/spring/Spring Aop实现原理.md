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

findCandidateAdvisors方法中做了下面这些事情：

1. 找出所有的beanName
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

