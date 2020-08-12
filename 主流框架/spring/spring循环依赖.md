# spring循环依赖

循环依赖根据注入的时机分成两种类型：

-  **构造器循环依赖**。依赖的对象是通过构造器传入的，发生在实例化 Bean 的时候。
-  **设值循环依赖**。依赖的对象是通过 setter 方法传入的，对象已经实例化，发生属性填充和依赖注入的时候。

对于构造器循环是没法解决的，因为是发生在实例化的时候。

对于设置值的情况，因为是发生在属性填充的时候，所以可以使用一些方法来实现。spring中就只有set方式注入，并且是单例的才可以实现循环依赖。

## 单例-构造器注入依赖

我们的配置是这样的：

```xml
	<beans>
		<bean id="circleB" class="org.springframework.CircleB" scope="prototype">
			<constructor-arg>
				<ref bean="circleA"/>
			</constructor-arg>
		</bean>
		<bean id="circleA" class="org.springframework.CircleA"  scope="prototype">
			<constructor-arg>
				<ref bean="circleB"/>
			</constructor-arg>
		</bean>
	</beans>
```

在doGetBean的时候，会先调用getSingleton(beanName)获取缓存的数据，如果获取到缓存的数据就直接返回，不用去新建了，这也是解决循环依赖的关键，我们后面会讲到。其中这个缓存是在实例化之后才会放进去的，而构造函数注入就发生在实例化的时候，这个时候没有这个缓存，所以单例构造函数注入没法解决循环依赖。

好了，来看看单例构造函数注入的流程：

在doCreateBean()的时候，会调用createBeanInstance()初始化实例，初始化的时候是通过构造函数使用反射生成实例。所以会去解析构造函数的参数 - autowireConstructor()，如果使用了ref参数，实际上就会拿着ref的值作为beanName去获取bean。

在getSingleton()方法中会调用beforeSingletonCreation()方法去做检查。

可以看到每次getSingleton的时候，会先在singletonsCurrentlyInCreation加入当前bean，如果之前已经存在的话，就会报错。

```java
if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
	throw new BeanCurrentlyInCreationException(beanName);
}

// singletonsCurrentlyInCreation的结构是这样的。
/** Names of beans that are currently in creation */
private final Set<String> singletonsCurrentlyInCreation =      Collections.newSetFromMap(new ConcurrentHashMap<>(16));
```

现在我们假设类A和类B的构造函数相互依赖。

如果类A构造函数需要类B，在生成类A的时候，就会在singletonsCurrentlyInCreation里面加入A，然后生成

A的实例，A的构造函数需要B，所以要去生成B。

生成B的时候，同样的过程，也会在singletonsCurrentlyInCreation里面加入B，然后再去解析B的构造函数，需要A，再去生成A，在singletonsCurrentlyInCreation发现已经有A了，就抛出异常。

## 原型-循环依赖

对于原型的scope，都不允许循环依赖。

在doGetBean的时候，会先调用isPrototypeCurrentlyInCreation方法判断原型模式是否在创建。

```java
	protected boolean isPrototypeCurrentlyInCreation(String beanName) {
		Object curVal = this.prototypesCurrentlyInCreation.get();
		return (curVal != null &&
				(curVal.equals(beanName) || (curVal instanceof Set && ((Set<?>) curVal).contains(beanName))));
	}
```

对于原型模式，在创建bean的时候会先调用beforePrototypeCreation(beanName)，如果prototypesCurrentlyInCreation没有值，就给他设置进去。

```java
	protected void beforePrototypeCreation(String beanName) {
		Object curVal = this.prototypesCurrentlyInCreation.get();
		if (curVal == null) {
			this.prototypesCurrentlyInCreation.set(beanName);
		}
		else if (curVal instanceof String) {
			Set<String> beanNameSet = new HashSet<>(2);
			beanNameSet.add((String) curVal);
			beanNameSet.add(beanName);
			this.prototypesCurrentlyInCreation.set(beanNameSet);
		}
		else {
			Set<String> beanNameSet = (Set<String>) curVal;
			beanNameSet.add(beanName);
		}
	}
// prototypesCurrentlyInCreation变量声明如下：
/** Names of beans that are currently in creation */
private final ThreadLocal<Object> prototypesCurrentlyInCreation =      new NamedThreadLocal<>("Prototype beans currently in creation");
```

可以看到，如果只有一个beanName,放的是String，如果有多个，就放Set<String\>的类型。

获取到bean之后，会调用afterPrototypeCreation(beanName)来移除。

### 原型构造函数注入

来具体看一下原型构造函数注入的过程。

将前面的配置改成scope="prototype"，就变成了原型构造器注入的方式。

对于构造函数注入的方式，根据前面的内容，我们知道通过构造函数实例化的时候会递归调用doGetBean方法来生成，先加入A，prototypesCurrentlyInCreation里面存的是string,再加入B的时候，就变成Set<String\>，里面有A和B，解析B的时候，需要先获取A，发现prototypesCurrentlyInCreation已经有A了，报错。

### 原型设置值注入

对于设置值的方式，我们可以使用property的方式来看一下。

```

	<beans>
		<bean id="circleB" class="org.springframework.CircleB" scope="prototype">
			<property name="circleA" ref="circleA" ></property>
		</bean>
		<bean id="circleA" class="org.springframework.CircleA"  scope="prototype">
			<property name="circleB" ref="circleB" ></property>
		</bean>
	</beans>
```

在doCreateBean的时候，实例化完成之后，会去调用populateBean()去填充属性，对于我们这种配置方式，回去调用applyPropertyValues方法，来完成属性的填充，由于我们属性是通过“ref”指向另一个bean，所以会去调用beanFactory.getBean(refName)去获取引用的bean，这个过程会去递归调用doGetBean去生成相关的bean，所以就会在prototypesCurrentlyInCreation加入值，之后发现重复就会产生报错。

## 单例设置值循环依赖解决方案

在doCreateBean方法中会判断是否需要提前暴露

```java
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
```

其中isSingletonCurrentlyInCreation()就是判断singletonsCurrentlyInCreation里面是否存在这个值。之前提到过，在创建bean之前会先调用beforeSingletonCreation(beanName)在singletonsCurrentlyInCreation里面加入值。

```java
		if (earlySingletonExposure) {
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
```

addSingletonFactory里面其实就是在singletonFactories和registeredSingletons加入当前bean，在earlySingletonObjects里面移除当前bean。

```java
	protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}
```

而在doGetBean方法中，会先执行下面这个方法，从缓存中先获取

```java
// Eagerly check singleton cache for manually registered singletons.	
Object sharedInstance = getSingleton(beanName);
```

看一下怎么获取的

```java
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

首先先从singletonObjects获取，如果拿到了，就说明已经存在了完整的单例bean了，如果没有获取到并且当前bean正在创建，则从earlySingletonObjects获取，如果没有获取到，再从singletonFactories获取singletonFactory去获取bean。这个singletonFactory是我们之前加入的，getObject方法实际是调用getEarlyBeanReference，得到我们之前暴露的bean实例。