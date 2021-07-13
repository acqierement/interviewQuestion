# Spring Boot的自动化配置原理

## 启动过程

先简单说一下启动的过程，从run开始。

```java
context = createApplicationContext();
prepareContext(context, environment, listeners, applicationArguments, printedBanner);
refreshContext(context);
```

先获取applicationContext。然后prepareContext里面去加载bean。最后再去执行applicationContext的refresh方法。这些都是spring的基本内容。

## 自动配置

启动类有个注解@SpringBootApplication，这个注解上面有@EnableAutoConfiguration，其中引入了一个类

```java
@Import({AutoConfigurationImportSelector.class})
```

这个类有一个主要方法：

```java
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
```

getAutoConfigurationEntry方法里面调用了getCandidateConfigurations方法

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

loadFactoryNames里面最后有这么一个语句：

```java
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
```

这个常量是META-INF/spring.factories。

所以我们知道了，springboot会去加载各个包下面的META-INF/spring.factories配置文件。

但是获取到配置后又是怎么跟spring进行整合的呢？

## 配置解析

主要是通过ConfigurationClassPostProcessor这个类来对加了@Configuration注解的类进行解析

```java
public class ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor,
		PriorityOrdered, ResourceLoaderAware, BeanClassLoaderAware, EnvironmentAware {
	......
}
```

这个类实现了BeanDefinitionRegistryPostProcessor，所以会调用其postProcessBeanDefinitionRegistry方法。具体调用地方是在refresh的invokeBeanFactoryPostProcessors()方法。

```java
	@Override
	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
		......
		processConfigBeanDefinitions(registry);
	}
```

在这里面会判断BeanDefinition是否为配置类，如果是，则用解析类对其解析，其实就是ConfigurationClassParser.parse()方法

在这个方法里面，会对@PropertySource、@ComponentScan、@Import、@ImportResource、@Bean等进行解析。我们这里只看一下@Import的解析。

首先会先拿到该类及其父类所有的@Import引入的类。

然后在processImports方法里，对其进行处理

```java
				for (SourceClass candidate : importCandidates) {
					if (candidate.isAssignable(ImportSelector.class)) {
						// Candidate class is an ImportSelector -> delegate to it to determine imports
						Class<?> candidateClass = candidate.loadClass();
						ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
								this.environment, this.resourceLoader, this.registry);
                        // 将select存起来
						if (selector instanceof DeferredImportSelector) {
							this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
						}
                        ......
					}
                }
```

这里面其实就拿到了我们import进去的AutoConfigurationImportSelector类，可以看到使用了一个handler进行处理

**ConfigurationClassParser.DeferredImportSelectorHandler.handle()**

```java
@Nullable
private List<DeferredImportSelectorHolder> deferredImportSelectors = new ArrayList<>();

public void handle(ConfigurationClass configClass, DeferredImportSelector importSelector) {
   DeferredImportSelectorHolder holder = new DeferredImportSelectorHolder(configClass, importSelector);
   if (this.deferredImportSelectors == null) {
      DeferredImportSelectorGroupingHandler handler = new DeferredImportSelectorGroupingHandler();
      handler.register(holder);
      handler.processGroupImports();
   }
   else {
      this.deferredImportSelectors.add(holder);
   }
}
```

把这个select放到了deferredImportSelectors这个list中。

紧接着parse的后面，就调用了this.deferredImportSelectorHandler.process()方法。也就是刚把import解析出来，就调用其process方法。

```java
public void process() {
   List<DeferredImportSelectorHolder> deferredImports = this.deferredImportSelectors;
   this.deferredImportSelectors = null;
   try {
      if (deferredImports != null) {
         DeferredImportSelectorGroupingHandler handler = new DeferredImportSelectorGroupingHandler();
         deferredImports.sort(DEFERRED_IMPORT_COMPARATOR);
         deferredImports.forEach(handler::register);
         handler.processGroupImports();
      }
   }
   finally {
      this.deferredImportSelectors = new ArrayList<>();
   }
}
```

这里首先将每个select注册到DeferredImportSelectorGroupingHandler中。

**ConfigurationClassParser.DeferredImportSelectorGroupingHandler#register**

```java
private final Map<Object, DeferredImportSelectorGrouping> groupings = new LinkedHashMap<>();

private final Map<AnnotationMetadata, ConfigurationClass> configurationClasses = new HashMap<>();

public void register(DeferredImportSelectorHolder deferredImport) {
   Class<? extends Group> group = deferredImport.getImportSelector().getImportGroup();
   DeferredImportSelectorGrouping grouping = this.groupings.computeIfAbsent(
         (group != null ? group : deferredImport),
         key -> new DeferredImportSelectorGrouping(createGroup(group)));
   grouping.add(deferredImport);
   this.configurationClasses.put(deferredImport.getConfigurationClass().getMetadata(),
         deferredImport.getConfigurationClass());
}
```

这里通过AutoConfigurationImportSelector#getImportGroup方法拿到了AutoConfigurationGroup这个内部类，包装成了DeferredImportSelectorGrouping，然后将其保存在groupings里面。

紧接着调用ConfigurationClassParser.DeferredImportSelectorGroupingHandler#processGroupImports对其进行处理

```java
public void processGroupImports() {
    for (DeferredImportSelectorGrouping grouping : this.groupings.values()) {
        Predicate<String> exclusionFilter = grouping.getCandidateFilter();
        grouping.getImports().forEach(entry -> {
            ConfigurationClass configurationClass = this.configurationClasses.get(entry.getMetadata());
            try {
                processImports(configurationClass, asSourceClass(configurationClass, exclusionFilter),
                               Collections.singleton(asSourceClass(entry.getImportClassName(), exclusionFilter)),
                               exclusionFilter, false);
            }
        });
    }
}
```

grouping.getImports()会去调用group.process对每个select进行处理

**AutoConfigurationImportSelector.AutoConfigurationGroup#process**

```java
		@Override
		public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
			AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
					.getAutoConfigurationEntry(annotationMetadata);
			this.autoConfigurationEntries.add(autoConfigurationEntry);
			for (String importClassName : autoConfigurationEntry.getConfigurations()) {
				this.entries.putIfAbsent(importClassName, annotationMetadata);
			}
		}
```

**AutoConfigurationImportSelector#getAutoConfigurationEntry**

```java
	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```



```
DeferredImportSelectorGrouping
```





```java
((AbstractApplicationContext) applicationContext).refresh();
```

所以最后还是调用AbstractApplicationContext的refresh方法。

在invokeBeanFactoryPostProcessors里面的invokeBeanDefinitionRegistryPostProcessors()方法会分别调用后置处理器的postProcessBeanDefinitionRegistry方法。

其中有ConfigurationClassPostProcessor这个后置处理器。该类实现了BeanDefinitionRegistryPostProcessor这个接口，所以就会执行postProcessBeanDefinitionRegistry方法，该方法里面调用了ConfigurationClassParser.parse方法，

传入的参数类型是多个BeanDefinitionHolder，就包括启动类的BeanDefinitionHolder。

```java
	public void parse(Set<BeanDefinitionHolder> configCandidates) {
		for (BeanDefinitionHolder holder : configCandidates) {
			BeanDefinition bd = holder.getBeanDefinition();
			try {
				if (bd instanceof AnnotatedBeanDefinition) {
					parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
				}
				else if (bd instanceof AbstractBeanDefinition && ((AbstractBeanDefinition) bd).hasBeanClass()) {
					parse(((AbstractBeanDefinition) bd).getBeanClass(), holder.getBeanName());
				}
				else {
					parse(bd.getBeanClassName(), holder.getBeanName());
				}
		}

		this.deferredImportSelectorHandler.process();
	}
```

启动类会进到第一个if

```java
parse(((AnnotatedBeanDefinition) bd).getMetadata(), holder.getBeanName());
```

入参包括类的注解，对注解进行解析

这边会去递归地解析注解

```java
	protected final void parse(AnnotationMetadata metadata, String beanName) throws IOException {
		processConfigurationClass(new ConfigurationClass(metadata, beanName));
	}
```

processConfigurationClass方法中：

sourceClass = doProcessConfigurationClass(configClass, sourceClass);

```java
	protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
			throws IOException {

		// Process any @Import annotations
		processImports(configClass, sourceClass, getImports(sourceClass), true);
        
		// No superclass -> processing is complete
		return null;
	}
```

这里的getImports方法其实就是递归地去获取@Import的值。其中就有AutoConfigurationImportSelector类

processImports里面有

```java
					if (candidate.isAssignable(ImportSelector.class)) {
						// Candidate class is an ImportSelector -> delegate to it to determine imports
						Class<?> candidateClass = candidate.loadClass();
						ImportSelector selector = ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class,
								this.environment, this.resourceLoader, this.registry);
						if (selector instanceof DeferredImportSelector) {
							this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
						}
```

这里如果是DeferredImportSelector的子类，会执行handle方法

```java
this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector) selector);
```

这里面会把AutoConfigurationImportSelector类加入到deferredImportSelectorHandler中。

然后调用前面parse方法里面的this.deferredImportSelectorHandler.process();

```java
		public void process() {
			List<DeferredImportSelectorHolder> deferredImports = this.deferredImportSelectors;
			this.deferredImportSelectors = null;
			try {
				if (deferredImports != null) {
					DeferredImportSelectorGroupingHandler handler = new DeferredImportSelectorGroupingHandler();
					deferredImports.sort(DEFERRED_IMPORT_COMPARATOR);
					deferredImports.forEach(handler::register);
					handler.processGroupImports();
				}
			}
			finally {
				this.deferredImportSelectors = new ArrayList<>();
			}
		}

	}
```

deferredImports.forEach(handler::register)里面会将AutoConfigurationImportSelector加入到this.groupings中

然后执行后面的handler.processGroupImports()方法

```java
		public void processGroupImports() {
			for (DeferredImportSelectorGrouping grouping : this.groupings.values()) {
				grouping.getImports().forEach(entry -> {
					ConfigurationClass configurationClass = this.configurationClasses.get(
							entry.getMetadata());

						processImports(configurationClass, asSourceClass(configurationClass),
								asSourceClasses(entry.getImportClassName()), false);

				});
			}
		}
```

这里遍历了this.groupings的每个类，通过getImprots方法获取配置信息，这里就回到了AutoConfigurationImportSelector的getImports()方法。

这样我们就完成了springboot获取配置文件的分析。