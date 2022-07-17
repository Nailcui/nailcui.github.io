```
getOrCreateEnvironment
configureEnvironment
environmentPrepared

createApplicationContext
prepareContext
	context.setEnvironment
	postProcessApplicationContext
	applyInitializers
	listeners.contextPrepared
refreshContext
afterRefresh

```



构造方法 > postConstruct > afterPropertiesSet(InitializingBean) > init方法

`AbstractAutowiredCapableBeanFactory`类中的`invokeInitMethods` 控制初始化相关的逻辑





```
@Bean
@DependsOn

@Conditional
@ConditionalOnClass
@ConditionalOnMissingClass
@ConditionalOnBean
@ConditionalOnMissingBean
@ConditionalOnProperty
@ConditionalOnResource
@ConditionalOnAvailableEndpoint
@ConditionalOnCloudPlatform
@ConditionalOnSingleCandidate
@ConditionalOnWebApplication
@ConditionalOnNotWebApplication

@Import
@ImportAutoConfiguration

@Configuration
@EnableConfigurationProperties



```



EventListener

```
EventListener
     ↑
ApplicationListener
     ↑
SmartApplicationListener
     ↑
GenericApplicationListener
     ↑
GenericApplicationListenerAdapter
```



spring boot 扩展点

```
ApplicationContextInitializer
BeanDefinitionRegistryPostProcessor
EnvironmentPostProcessor
BeanFactoryPostProcessor

```



### Spring 扩展点

####EnvironmentPostProcessor

事件`ApplicationEnvironmentPreparedEvent` 触发到`ConfigFileApplicationListener`之后，执行此钩子



#### ApplicationContextInitializer

> 上下文刷新之前，prepareContext中，applyInitializers的时候执行

在`ConfigurableApplicationContext`类型（或者子类型）的`ApplicationContext`做`refresh`之前，允许我们对`ConfiurableApplicationContext`的实例做进一步的设置和处理。

```java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {

	/**
	 * Initialize the given application context.
	 * @param applicationContext the application to configure
	 */
	void initialize(C applicationContext);
}
```

这时候spring容器还没被初始化，所以想要自己的扩展的生效，有以下三种方式：

- 在启动类中用`springApplication.addInitializers(new TestApplicationContextInitializer())`语句加入

- 配置文件配置`context.initializer.classes=com.example.MyApplicationContextInitializer`

- SPI方式，在`spring.factories`中加入`org.springframework.context.ApplicationContextInitializer=com.example.MyApplicationContextInitializer`



#### BeanDefinitionRegistryPostProcessor

这个接口在读取项目中的`beanDefinition`之后执行

使用场景：你可以在这里动态注册自己的`beanDefinition`，可以加载`classpath`之外的`bean`

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
  void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry var1) throws BeansException;
}
```



#### BeanFactoryPostProcessor

这个接口是`beanFactory`的扩展接口，调用时机在spring在读取`beanDefinition`信息之后，实例化bean之前。

在这个时机，用户可以通过实现这个扩展接口来自行处理一些东西，比如修改已经注册的`beanDefinition`的元信息。



#### InstantiationAwareBeanPostProcessor



#### SmartInstantiationAwareBeanPostProcessor



#### BeanFactoryAware

这个类只有一个触发点，发生在bean的实例化之后，注入属性之前，也就是Setter之前。这个类的扩展点方法为setBeanFactory，可以拿到BeanFactory这个属性。

使用场景为，你可以在bean实例化之后，但还未初始化之前，拿到 BeanFactory，在这个时候，可以对每个bean作特殊化的定制。也或者可以把BeanFactory拿到进行缓存，日后使用。



#### BeanPostProcessor

bean对象初始化前后调用

```java
public interface BeanPostProcessor {

	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
}
```



#### ApplicationContextAwareProcessor



#### BeanNameAware



#### PostConstruct



#### InitializingBean



#### FactoryBean





