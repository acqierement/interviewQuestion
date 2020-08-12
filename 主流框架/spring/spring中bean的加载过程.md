# spring中bean的加载过程

可参考[图文并茂，揭秘 Spring 的 Bean 的加载过程](https://www.jianshu.com/p/9ea61d204559)

## 1.转换beanName

1. 对于指向factory bean会加上& 前缀，如果有，去掉这个前缀。
2. 处理别名，得到真正的名字。

## 2.合并RootBeanDefinition

主要在方法中：

```java
RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
```

如果一个类有父类，要获取完整的信息，就要把所有的父类信息拿到。这里通过递归去获取父类的信息。

```java
String parentBeanName = transformedBeanName(bd.getParentName());
if (!beanName.equals(parentBeanName)) {
    pbd = getMergedBeanDefinition(parentBeanName);
}
```

拿到父类的信息之后，进行下面的代码：

```java
// Deep copy with overridden values.
mbd = new RootBeanDefinition(pbd);
mbd.overrideFrom(bd);
```

深克隆父类得到RootBeanDefinition，再用缓存的beanDefinition里的信息覆盖RootBeanDefinition的信息。也就是以缓存中的beanDefinition信息为准。

这样就得到了一个完整的RootBeanDefinition。

## 3.实例化bean

实例化bean都会到这个方法。

```java
beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, this);
```

真正的实例化：

```

if(!bd.hasMethodOverrides()){
	// 如果没有方法重写，通过构造函数使用反射来实例化对象
	......
	return BeanUtils.instantiateClass(constructorToUse);
} else {
	// 因为用户使用了 replace 和 lookup 的配置方法，用到了动态代理加入对应的逻辑
	// Must generate CGLIB subclass.
	return instantiateWithMethodInjection(bd, beanName, owner);
}

```

## 4.属性填充

根据给的bean definition里面的属性信息来填充实例instanceWrapper

```java
populateBean(beanName, mbd, instanceWrapper);
```

在填充之前，会调用postProcessAfterInstantiation来进行处理

```java
		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
						return;
					}
				}
			}
		}
```

然后才是填充属性

```
		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// 根据名称注入
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// 根据类型注入
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}
```

再调用InstantiationAwareBeanPostProcessor.postProcessPropertyValues进行处理

```java
if (hasInstAwareBpps) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            pvs = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
            if (pvs == null) {
                return;
            }
        }
    }
}
```

最后再应用属性值    applyPropertyValues(beanName, mbd, bw, pvs);

可以看到这里的处理过程，应用 InstantiationAwareBeanPostProcessor 处理器，在属性注入前后进行处理。**假设我们使用了 @Autowire 注解，这里会调用到 AutowiredAnnotationBeanPostProcessor 来对依赖的实例进行检索和注入的**，它是 InstantiationAwareBeanPostProcessor 的子类。


## 5.初始化

### 触发Aware

会先调用invokeAwareMethods()方法

```java
if (bean instanceof Aware) {
	if (bean instanceof BeanNameAware) {
		((BeanNameAware) bean).setBeanName(beanName);
	}
	if (bean instanceof BeanClassLoaderAware) {
		ClassLoader bcl = getBeanClassLoader();
		if (bcl != null) {
			((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
		}
	}
	if (bean instanceof BeanFactoryAware) {
		((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
	}
}
```

可以看到如果实现了各个aware接口，会在这里设置相应的数据。

### 执行初始化方法

之后会分别调用下面的方法

```java
// 初始化前调用
wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 // 触发自定义 init 方法
invokeInitMethods(beanName, wrappedBean, mbd);
// 初始化后调用
wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

```

invokeInitMethods方法会先判断是否实现了InitializingBean接口，如果有会调用**afterPropertiesSet**方法。

如果有自定义的初始化方法，会执行自己定义的初始化方法。

## 6.类型转化

```java
if (requiredType != null && !requiredType.isInstance(bean)) {
    T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
    if (convertedBean == null) {
        throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
    }
    return convertedBean;
｝
```

