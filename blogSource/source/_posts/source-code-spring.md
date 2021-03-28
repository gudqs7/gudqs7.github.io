---
title: Spring 源码笔记
date: 2021-01-23 18:28:28
top: true
tags: 
  - spring
categories:
  - [java]
  - [spring]
  - [source-code]

---



# Spring源码分析

## 关键类介绍

### ApplicationContext

```bash
万能的 applicationContext, 但实际上各种能力都是依赖于其他的类, 比如 getBean 是 beanFactory 的, publishEvent 是事件广播器的, 等等. 其本身是一个综合体, 整合这些能力, 便于开发者调用和理解.

# 下面列一下相关的接口, 抽象类, 和具体类
ApplicationContext
	是一个只读的 bean 容器
	可以加载解析配置文件(如xml)
	可以发布事件和注册监听
	具有国际化消息处理能力
ConfigurableApplicationContext
	是一个具有可配置能力的 容器(可设置各个参数, 如id, 父容器)
	具有容器生命周期概念, 如启动,停止,关闭.
AbstractApplicationContext
	模板方法模式的抽象类, 定义了容器的模板(refresh方法), 但由具体的子类实现部分方法
	管理Bean和BeanFactory的PostProcessor
	管理事件的监听和处理
AbstractRefreshableApplicationContext
	为可重复刷新的容器提供基类
	加入了BeanFactory的管理(创建/关闭等)
AbstractRefreshableConfigApplicationContext
	加入了configLocation字段, 用于某些容器初始化BeanFactory和Bean
AbstractXmlApplicationContext
	定义了读取xml配置文件来加载BeanFactory的代码, 使得子类只需提供配置文件地址或Resource
ClassPathXmlApplicationContext
	继承基类, 提供配置文件地址的构造方法, 调用refresh加载BeanFactory
		
```

### BeanFactory

```bash
1.核心中的核心, 加载和管理 beanDefinitions(Bean配置信息), 创建和管理 bean 对象实例, 注册和管理 BeanPostProcessor(Bean扩展)

# 下面列一下相关的接口, 抽象类, 和具体类
BeanFactory
  定义了 Bean 的基础操作接口, 如 getBean, getType, isSingleton 等

SingletonBeanRegistry
  定义了单例对象的操作接口 (注册/获取/是否已存在) 

HierarchicalBeanFactory
  定义了父 BeanFactory 的相关操作接口(获取)

ConfigurableBeanFactory
  定义了对 BeanFactory 做各种配置的操作接口, 包括 BeanPostProcessor, setParentBeanFactory, destroyBean, registerAlias, resolveAliases 等

DefaultSingletonBeanRegistry
  实现了 SingletonBeanRegistry 接口, 即实现了单例对象的缓存管理, 包括一级/二级/三级(二级三级只依赖循环用上的两个缓存)
  
FactoryBeanRegistrySupport
  继承了 DefaultSingletonBeanRegistry
	实现对使用 FactoryBean 存储和获取 bean 对象实例方式的支持
	
AbstractBeanFactory
  继承了 FactoryBeanRegistrySupport
  实现了 BeanFactory/HierarchicalBeanFactory/ConfigurableBeanFactory 定义的接口
  实现了具体 getBean, 包括缓存管理等
  
AutowireCapableBeanFactory
  定义了根据 class 类型获取 BeanDefinition 信息以及 Bean 对象的接口

AbstractAutowireCapableBeanFactory
  继承自 AbstractBeanFactory 
  实现了 AutowireCapableBeanFactory 中定义的方法(就是实现了根据 class 获取 bean 或 BeanDefinition)
  实现了 createBean, 也就是真正的实例化一个对象的过程, 包括实例化, 为需要赋值的字段注入相应的值
  同时触发了 BeanPostProcessor 的方法调用
  
BeanDefinitionRegistry
  定义了 BeanDefinition 的注册/获取/移除
  
ListableBeanFactory
	定义了 BeanDefinition 的可遍历性
  
ConfigurableListableBeanFactory
  结合 ListableBeanFactory 和 ConfigurableBeanFactory 并补充完善了几个相关接口 (如 getBeanNamesIterator 接口 )

DefaultListableBeanFactory
  继承了 AbstractAutowireCapableBeanFactory
  实现了 BeanDefinitionRegistry/ConfigurableListableBeanFactory 的接口
 
# 总结:
定义处:
BeanFactory(getBean)
SingletonBeanRegistry(addSingleton)
HierarchicalBeanFactory(getParentBeanFactory)
ConfigurableBeanFactory(addBeanPostProcessor)
AutowireCapableBeanFactory(autowireBean)
BeanDefinitionRegistry(registerBeanDefinition)
ListableBeanFactory(getBeanDefinitionNames)
ConfigurableListableBeanFactory(getBeanNamesIterator)

实现处:
DefaultSingletonBeanRegistry(registerSingleton)
FactoryBeanRegistrySupport(getObjectFromFactoryBean)
AbstractBeanFactory(doGetBean)
AbstractAutowireCapableBeanFactory(createBean)
DefaultListableBeanFactory(registerBeanDefinition)
```





## 容器初始化过程

```bash
1.setParent(): 处理父容器 
2.setConfigLocations(): 解析并设置xml配置文件路径
3.refresh(): 创建 beanFactory 对象并初始化, 读取 xml 配置文件得到 beanDefinitions, 接着处理两种 PostProcessor, 然后添加国际化处理器和事件广播器以及相应的初始化和一些处理, 最后实例化单例的 bean 等等.

#外圈结束, 再看 refresh() 里面的每个方法
1.prepareRefresh(): 准备工作, 一些字段值的设置和处理.
2.obtainFreshBeanFactory(): 创建一个 beanFactory 对象并注册到 applicationContext (即赋值到字段上), 然后解析 xml 配置文件(或注解配置)的信息, 解析得到 beanDefinitions 并注册到容器中.
3.然后是一些对 beanFactory 对象的完善配置的代码
4.扫描并执行 BeanFactoryPostProcessor(其作用是为beanFactory对象添加东西提供扩展性), 其中我认识的就只有 ConfigurationClassPostProcessor(这个类作用就是解析 @Configuration/@Component/@Import/@ImportSource/@ComponentScan等基础注解).
5.扫描实现了 BeanPostProcessor 接口的 bean 并注册到 beanFactory 中存起来, 等 createBean 创建对象时会在对应的时机执行一些对应的方法(钩子). 常见的各种 XxxAware 就是靠这个实现的咯.
6.接着, 初始化国际化资源处理器, 事件广播器, 并注册一些需要注册的事件(也注册容器内实现对应接口的 bean)
7.处理一些 beanFactory 的配置, 接着为所有单例且非懒加载的(不就是默认策略嘛) bean 创建实例, 缓存起来.
8.广播容器加载完成了的事件. 以及处理生命周期.
```

> 最后总结下, 先创建容器, 再将根据配置文件解析得到 BeanDefinition 注册到容器中, 然后处理两大扩展(BeanFactoryPostProcessor/BeanPostProcessor), 接着是Spring的国际化, 以及相当有用的事件广播器, 最后实例化 bean. 整体感觉其实很简单, 但其实有大量的工作交给了 BeanPostProcessor.



### 超长源码分析过程

```java
// 先随便写个 main 方法, 如我写的, 可测试依赖循环问题和事件监听:
// 包名: cn.gudqs7.spring.tests, 改动则需同步修改xml哦
// 进入对应类代码: 快捷键 Cmd+Option+鼠标点击 (或 Ctrl+Alt+鼠标左键 ); 如果是接口松开 Option(或Alt)键

Test.java
public class Test {

	public static void main(String[] args) {
		ApplicationContext xmlContext = new ClassPathXmlApplicationContext("application-wq.xml");
		UserServiceImpl userService = xmlContext.getBean(UserServiceImpl.class);
		userService.sayHi();
	}

}


application-wq.xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd" default-autowire="byName">

	<bean name="userService" class="cn.gudqs7.spring.tests.UserServiceImpl">
		<property name="starter"><ref bean="serverStarter"/></property>
	</bean>
	<bean name="serverStarter" class="cn.gudqs7.spring.tests.ServerStarter">
		<property name="userService"><ref bean="userService"/></property>
	</bean>

</beans>

    
UserServiceImpl.java
@Service
public class UserServiceImpl {

	@Resource
	private ServerStarter starter;

	public void sayHi() {
		System.out.println(starter);
		System.out.println("Hello Spring!");
	}

	public void setStarter(ServerStarter starter) {
		this.starter = starter;
	}
}


ServerStarter.java
@Service
public class ServerStarter implements ApplicationListener<ContextRefreshedEvent> {

	@Inject
	private UserServiceImpl userService;

	@Override
	public void onApplicationEvent(ContextRefreshedEvent event) {
		String applicationName = event.getApplicationContext().getApplicationName();
		System.out.println(applicationName);
		System.out.println(userService);
		System.out.println("========== started by gudqs7 ==============");
		System.out.println("========== started by gudqs7 ==============");
		System.out.println("========== started by gudqs7 ==============");
	}

	public void setUserService(UserServiceImpl userService) {
		this.userService = userService;
	}
}
```



> 😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄

```java
// 接下来, 进入ClassPathXmlApplicationContext#ClassPathXmlApplicationContext(java.lang.String) 方法中

// 其跳转到了
  public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		// 为容器的 parent 字段赋值, 若 parent 不为空, 且有 ConfigurableEnvironment, 则合并数据(将父容器有的加到子容器中)
		// 即执行了 org.springframework.context.support.AbstractApplicationContext.setParent()
		super(parent);

		// 为 configLocations 字段赋值(告知配置文件位置), 赋值前会根据环境变量解析(此时环境变量中只有系统环境变量: 如JAVA_HOME).
		setConfigLocations(configLocations);
		if (refresh) {
			// 注释在下面
      refresh();
		}
	}
  
  
```

> 😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄

```java
// 然后具体的看 refresh 方法
public void refresh() throws BeansException, IllegalStateException {
		// 1.设置容器初始的一些属性(时间,状态)，初始化占位符数据源并校验所有 bean 所使用的占位符是否存在, 清空事件和监听
		// 2.清空重置旧的 beanFactory, 再创建新的 beanFactory 并通过解析 xml或注解 加载 beanDefinitions
		// 3.设置了一些 beanFactory 的属性, 添加了几个有用的 BeanPostProcessor, 还添加了几个 bean 到容器中(都是环境相关的 bean)
		// 4.子类对beanFactory 添加自己的特殊的 BeanPostProcessor (如servletContxt/servletConfig注入)
		// 5.扫描容器中实现了 BeanFactoryPostProcessor 接口的 bean 将其注册到 beanFactory 中并执行
		// 6.扫描容器中实现了 BeanPostProcessor 接口的 bean 将其注册到 beanFactory 但不执行(实例化 bean 对象那会有几个执行时机)
		// 7.创建一个国际化资源解析器并注册到 beanFactory; 创建一个事件广播器并注册到 beanFactory.
		// 8.调用子类的其他刷新时需要做的事情(模板方法)
		// 9.扫描容器中实现了 ApplicationListener 接口的 bean, 将其预存到广播器中但不执行
		//10.完成 beanFactory 的一些配置(包括终结一些东西, 如 setTempClassLoader(null) ); 将单例的 bean 创建出来放入容器中(未设置lazy-init=true)的 bean
		//11.广播 ContextRefreshedEvent 事件， 初始化LifeCycleProcessor及调用其 onRefresh 方法.

		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			// 设置容器初始的一些属性(时间,状态)，初始化占位符数据源并校验所有 bean 所使用的占位符是否存在, 清空事件和监听
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			// 清空重置旧的 beanFactory, 再创建新的 beanFactory 并解析 xml或注解 加载 beanDefinitions
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			// 设置了一些 beanFactory 的属性, 添加了几个有用的 BeanPostProcessor, 还添加了几个 bean 到容器中(都是环境相关的 bean)
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 子类对beanFactory 添加自己的特殊的 BeanPostProcessor (如servletContxt/servletConfig注入)
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				// 扫描容器中实现了 BeanFactoryPostProcessor 接口的 bean 将其注册到 beanFactory 中并执行
				//   扫描6次, 2(BeanDefinitionRegistry/BeanFactory) x 3(优先级:高/中/其他)
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				// 扫描容器中实现了 BeanPostProcessor 接口的 bean 将其注册到 beanFactory 但不执行; 扫描6次: 2(MergedBeanDefinitionPostProcessor/其他) x 3(优先级: 高/中/其他)
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				// 创建一个国际化资源解析器并注册到 beanFactory.
				initMessageSource();

				// Initialize event multicaster for this context.
				// 创建一个事件广播器并注册到 beanFactory.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				// 调用子类的其他刷新时需要做的事情(模板方法)
				onRefresh();

				// Check for listener beans and register them.
				// 将可能存在的 applicationListeners 注册到事件广播器中(新建时是不存在的),
				// 然后扫描容器中实现了 ApplicationListener 接口的 bean, 将其预存到广播器中但不执行
				// 将之前 publishEvent() 想广播的事件广播出去, 然后字段 earlyApplicationEvents 赋值为空(代表之后可立即广播)
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				// 完成 beanFactory 的一些配置(包括终结一些东西, 如 setTempClassLoader(null) )
				// 注册默认的表达式解析器(若无相应的bean存在)
				// 扫描容器中实现了 LoadTimeWeaverAware 接口的 bean, 并触发(getBean)之前注册过的 BeanPostProcessor
				// 将单例的 bean 创建出来放入容器中(未设置lazy-init=true)的 bean
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				// 广播 ContextRefreshedEvent 事件， 初始化LifeCycleProcessor及调用其 onRefresh 方法.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				// 销毁缓存的单例对象
				destroyBeans();

				// Reset 'active' flag.
				// 变更状态
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				// 清空公共工具产生的缓存(内存松一口气).
				resetCommonCaches();
			}
		}
	}
```

> 我错了, 代码都放上去不如给个GitHub地址, 接下来省略代码吧, 只放注释😄😄😄😄😄😄😄😄😄😄😄😄😄😄😄

```java
// 挨个看里面的方法
org.springframework.context.support.AbstractApplicationContext#prepareRefresh
    // 1.设置容器初始的一些属性, 如启动时间, 当前状态
		// 2.打印开始日志
		// 3.初始化占位符数据源
		// 4.校验所有 bean 所使用的占位符是否存在
		// 5.清空事件和监听
		// Switch to active.
  
org.springframework.context.support.AbstractRefreshableApplicationContext#refreshBeanFactory
    // 1.存在旧的则先摧毁 bean 对象实例及缓存数据, 再将旧的置为 null
		// 2.创建一个新的 beanFactory 对象, 再设置 id及一些配置
		// 3.扫描并加载 beanDefinations
		// 4.设置这个新的 beanFactory 对象为 applicationContext 的 beanFactory 字段值.
  
org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory
    // 1.设置 beanFactory 的类加载器
		// 2.设置 beanFactory 的表达式解析器
		// 3.1:注册一个 BeanPostProcessor 用于将实现了ApplicationContext能力相关的Aware接口的 bean, 触发赋值setter 注入 applicationContext 对象
		// 3.2:设置 beanFactory 处理 bean 时要忽略的接口(主要是setter注入时忽视一些也是setter的方法, 因为这些方法会由 PostProcessor 来触发)
		// 4.注册一些特殊的 bean(注入这些bean时会注入 this 对象: 多功能工具人 ApplicationContext, 可见其和普通 bean 的注册方式不一样)
		// 5.注册一个 BeanPostProcessor 用于检测加载的 bean 是否实现了 ApplicationListener 接口, 若是, 则注册到事件广播器中(不是,是暂存在applicationListeners字段中, 等事件广播器创建后才注册)
		// 6.注册一个 BeanPostProcessor 用于触发实现了 LoadTimeWeaverAware 接口的 bean 的 setLoadTimeWeaver() 社会 LTW 实例.
		// 7.注册几个环境相关 bean 到容器中(Spring的环境对象, 以及系统环境变量和系统配置文件)

org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanFactoryPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, java.util.List<org.springframework.beans.factory.config.BeanFactoryPostProcessor>)
    // 1.若 beanFactory 实现了 BeanDefinitionRegistry 接口(new AnnotationConfigApplicationContext 就实现了)
		//    则扫描所有实现了 BeanDefinitionRegistryPostProcessor 接口的 bean, 根据优先级分三类(高/中/其他)依次执行
		// 2.然后扫描所有实现了 BeanFactoryPostProcessor 接口的 bean, 依旧是根据优先级分三类依次执行.
		// 3.每次执行前都会根据 Order 信息排序, 再遍历执行
  
org.springframework.context.support.PostProcessorRegistrationDelegate#registerBeanPostProcessors(org.springframework.beans.factory.config.ConfigurableListableBeanFactory, org.springframework.context.support.AbstractApplicationContext)
    // 1.扫描6次: 2(MergedBeanDefinitionPostProcessor/其他) x 3(优先级: 高/中/其他)
		//    将其加入到 beanFactory 的 beanPostProcessors 集合中
		// 2.再次加入 ApplicationListenerDetector (用于处理实现 ApplicationListener 的 bean 注册到事件广播器), 主要是使其在链末尾, 可以最后执行.

org.springframework.context.support.AbstractApplicationContext#registerListeners
    // 1.将可能存在的 applicationListeners 注册到事件广播器中(新建时是不存在的)
		// 2.扫描容器中实现了 ApplicationListener 接口的 bean, 将其预存到广播器中但不执行
		// 3.将之前 publishEvent() 想广播的事件广播出去, 然后字段 earlyApplicationEvents 赋值为空
		//    (因为 publishEvent() 中根据是否为空判断立刻执行或先存着) (另这也解释了 prepareRefresh() 中为何要赋值一个空集合)

  
org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization
    // 1.完成 beanFactory 的一些配置
		// 2.注册默认的表达式解析器(若无相应的bean存在)
		// 3.扫描容器中实现了 LoadTimeWeaverAware 接口的 bean, 并触发(getBean)之前注册过的 BeanPostProcessor
		// 4.将单例的 bean 创建出来放入容器中(未设置lazy-init=true)的 bean

org.springframework.context.support.AbstractApplicationContext#finishRefresh
    // 1.清空资源缓存
		// 2.创建一个生命周期管理器(start, refresh, stop等)并注册到 beanFactory
		// 3.触发生命周期管理器的 onRefresh()
		// 4.广播容器刷新完成的事件
		// 5.为 Spring Tool Suite 提供某些便捷(没用过, 不知道...)

```

> 可算复制完了, 如果你有幸直接跳读到这里, 那么送上地址 : [注意分支吧](https://github.com/gudqs7/spring-framework/tree/wq-comment) 
>
> 另外上面方法前带个 **#** 的, 复制到 IDEA 双击 Shift 然后粘贴, 选择 Symbols 搜索更准确呢!





## 获取容器对象过程

```bash
# 从getBean(Class type) 中进入
1.检查 applicationContext 和 beanFactory 的状态, 若有异常则给出准确的错误.
2.扫描容器中所有此 type 的 beanName, 遍历判断每个 beanName 是否可用
   可用则判断可用的个数是否刚好是一个, 是则直接调用 getBean() 返回对象实例
   若可用的个数超过一个, 则根据 beanDefinition 的 isPrimary 和对比配置的优先级是否为有最高的再返回最高的
   若都不行, 则报错.
3.接着看 getBean, 先试着从单例的缓存中获取, 若存在则返回.
4.若缓存中不存在, 则判断父容器是否存, 若存在则从父容器获取
   若父容器不存在, 则自己新建, 先标记 beanName 到 alreadyCreated 中(表示已经创建了防止重复创建) 再开始创建一个 bean.
5.创建一个新的 bean 实例, 先处理 beanDefinition 的 dependsOn 属性(即若存在则先调用 getBean 获取依赖的 bean)
6.若 beanDefinition 的设置是单例, 则通过闭包对创建对象前后进行一些异常处理和缓存处理(主要是彻底创建完后加入到单例一级缓存, 移除二级和三级缓存[循环依赖相关的两个缓存]).
7.通过反射根据 beanClass 创建一个对象实例, 然后将其添加到 singletonFactories 中(解决依赖循环问题)
8.调用 populateBean() 为对象的字段(属性)注入它所需要的值(可能是@Resource, @Value等); (此时可能会遇到依赖循环问题, 但解决这个问题的缓存在此之前就添加了, 所以不怕)
9.最后调用 initializeBean() 完成 bean 的初始化(调用 bean 的一些方法, 如 afterPropertiesSet), 返回对象实例.
```

> 总结: 先根据 type 找到 beanName, 找到后根据 beanName 创建对象; 创建对象前先检查缓存(单例), 再考虑父容器, 最后才是自己创建, 自己创建会先创建 dependsOn 的 bean 对象, 然后才通过反射实例化出一个对象实例(这里反射用到的class和构造方法, 通过实现 SmartInstantiationAwareBeanPostProcessor接口都可进行干预), 实例化后存到二级缓存, 再为字段赋值(注入); 最后调用 bean 的 init 相关的接口(如afterPropertiesSet), 就可以返回这个对象实例了.



### 超长源码分析过程

```java
// 从这个方法进入
@Override
public <T> T getBean(Class<T> requiredType) throws BeansException {
  // 判断容器的状态, 确保 beanFactory 可用.(主要是若不可用, 提示的错误信息会比getBeanFacgtory()中更准确)
  // 使用 beanFactory 的 getBean 方法获取对象并返回.
  assertBeanFactoryActive();
  return getBeanFactory().getBean(requiredType);
}

// 然后其他所有涉及的核心方法的注释
org.springframework.beans.factory.support.DefaultListableBeanFactory#resolveBean
    // 调用 resolveNamedBean, 如存在 bean 则直接返回. (核心)
		// 若不存在则从父容器中寻找, 父容器实现了 DefaultListableBeanFactory 则调与同子容器相同的方法
		// 若没实现则 通过 getBeanProvider 获取.
  
org.springframework.beans.factory.support.DefaultListableBeanFactory#resolveNamedBean(org.springframework.core.ResolvableType, java.lang.Object[], boolean)
    // 1.调用 getBeanNamesForType 获取所有与 type 相匹配的 beanName 集合.
		// 2.遍历判断每个 beanName 是否可用
		// 3.若可用的 beanName 只有一个, 则调用 getBean(beanName) 获取对象实例并返回
		//   若可用数超过一个, 则试着根据是否主要以及高优先级来确定一个 beanName 实例, 若能确定则返回, 不能则报错.

org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
    // 1.获取完整的 beanName
		// 2.调用 getSingleton1() 检查是否存在缓存, 这层检查可防止依赖循环.
		//     若存在, 则通过 getObjectForBeanInstance() 判断缓存的是 bean 还是 FactoryBean 并返回相应的对象实例.
		// 3.若不存在, 先试着从父容器获取(子容器不存在这个 beanDefinition 且父容器不为空)
		//     没有父容器则 调用 markBeanAsCreated() 标记这个 bean已经创建了 (先标记, 再创建)
		//     获取 beanDefinition, 判断其 dependsOn 属性是否存在, 存在则 先获取依赖的 bean
		//     调用 getSingleton2() 处理单例缓存
		// 4.而 getSingleton2() 中的闭包中 执行的 createBean() 方法中则才是创建实例并调用 BeanPostProcessor

org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, boolean)
    // 1.判断是否存在于 singletonObjects 中
		// 2.若不存在则判断 bean 是否处于创建中(未创建完成, 如循环依赖时)
		// 3.若处于创建中, 则同步后判断是否存在于 earlySingletonObjects (也就是 singletonFactories 移除后存入的地方)
		//      (因为FactoryBean占用空间大, 获取对象麻烦且速度更慢, 这是为了防止如果循环依赖链条很长 多次获取浪费CPU的问题)
		// 4.不存于 earlySingletonObjects 则代表第一次(也只会有一次)取 singletonFactories
		//    取出后调用 getObject() 并将其存入到 earlySingletonObjects, 然后从 singletonFactories 中移除. 以后就少走几行代码了.
  
org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String, org.springframework.beans.factory.ObjectFactory<?>)
    // 1.先确保是第一次创建单例对象, 防止重复创建
		// 2.进行一些异常处理
		// 3.调用 singletonFactory.getObject() 创建对象
		// 4.创建对象结束添加单例缓存和清空 singletonFactories / earlySingletonObjects 缓存.
  
org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
    // 1.调用 resolveBeanClass 解析得到真正的 bean class, 若解析不为空且处于某些情况下, 则复制一份 beanDefinition 并设置 beanClass 为解析所得
		// 2.执行 BeanPostProcessor 的 postProcessorsBeforeInstantiation() 方法
		// 3.调用 doCreateBean() 创建对象 并返回

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
    // 1.调用 createBeanInstance() 获得一个 对象实例的 包装类
		// 2.同步锁下执行 BeanFactoryPostProcessor 的 postProcessMergedBeanDefinition().
		// 3.添加 singletonFactories 缓存, 移除 earlySingletonObjects; 解决循环依赖问题.
		// 4.调用 populateBean() 检查字段是否需要注入对象实例, 是则获取对应的 bean 注入. (可能引起循环依赖)
		// 5.调用 initializeBean() 执行对象的一些 Aware 和 init 方法和 BeanPostProcessor 的 postProcessBeforeInitialization.
		// 6.最后返回对象实例.


```





## 各种实现的原理

```
1.为何我写的 class 实现一些接口(如ApplicationContextAware)后并放入容器中, 就可以获取到一些对象(如applicationContext)?
2.为何我写的 class 实现 ApplicationListener<XxxEvent> 后并放入容器中, 就能监听我想知道的事件?
3.为何Spring中遇到各种顺序问题, 只需要实现 Ordered 接口(或加上@Order注解)就能使其有序?
4.Spring是如何解决循环依赖的(指用字段注入而非构造方法)?
5.Spring可以用注解替换XML配置文件了, 是如何实现的呢(常用注解的实现原理)?
6.Spring AOP是如何实现的(指@Aspect)?
7.Spring 事务是如何实现的(指@Transaction)?
```

### 为何我写的 class 实现一些接口(如ApplicationContextAware)后并放入容器中, 就可以获取到一些对象(如applicationContext)?

```
1) 首先 AbstractApplicationContext#prepareBeanFactory 会添加一个ApplicationContextAwareProcessor
2) 这个 beanPostProcessor 负责在bean初始化之前注入context对象.
3) 这个 beanPostProcessor 的执行时机是在 doCreateBean 中的 postProcessBeforeInitialization()
```



### 为何我写的 class 实现 ApplicationListener<XxxEvent> 后并放入容器中, 就能监听我想知道的事件?

```
1) 在 AbstractApplicationContext#registerListeners() 中扫描容器内所有相关实现类加入到事件监听者集合中
2) 然后在publishEvent时，遍历事件监听者集合调用bean的方法即可。观察者模式！
3) 另外也用了BeanPostProcessor去实现, 叫 ApplicationListenerDetector, 加入时机同1, 执行时机同1.
4) 至于为何使用2种机制, 应该是因为 registerListeners() 时, 扫描只是当前的, 后续可能容器内的 bean 还会增加(我也猜不到啥形式增加, 反正简单写个类肯定不会), 所以还是需要 ApplicationListenerDetector 在这个 Bean 初始化时加入到监听者中去.
```



### 为何Spring中遇到各种顺序问题, 只需要实现 Ordered 接口(或加上@Order注解)就能使其有序?

> 因为 Spring 预先在执行这些东西之前, 进行一个排序动作, 然后才遍历执行. 包括AOP, BeanFactoryPostProcessor, BeanPostProcessor .

```
1) 比如说 BeanPostProcesser, 容器扫描后, 会像对bean集合排序, 再遍历执行.
2) 详细过程见 PostProcessorRegistrationDelegate#sortPostProcessors()
```



### Spring是如何解决循环依赖的(指用字段注入而非构造方法)?

```java
1) 首先， 假定有两个单例 bean A 和 B, A 持有 B, B 持有A， 构成循环
2) 此时程序调用getBean获取A，则在 doCreateBean 中 创建后将 bean 缓存到 singletonFactories 中
3) 然后设置属性B, 解析属性, 需要获取B对象
4) 获取B, 则执行doCreateBean 后执行解析属性, 需要获取 A对象 (又一次)
5) 获取A, 进入 doGetBean 中的 getSingleton, 此时判断 singletonFactories 中有A, 则可以直接取出A
6) 获得A后, 即可完成B的属性赋值, 然后会完成B的创建.
7) B创建完后, A就能获得B, 则A也完成了属性赋值, 最后完成创建A.
8) 到此, 返回即可.

> 总结: 首次获取A, 创建A对象后缓存一个存储A对象的 ObjectFactory 实例, 再解析属性时触发 getBean(B), 同理也会做缓存, 然后也解析属性, 触发getBean(A), 第二次获取A, 进入另一个逻辑, 返回 ObjectFactory 实例中存储的对象A, 即可完成getBean(A), 然后完成getBean(B), 再完成外层的getBean(A).  
  
TIPS:
步骤4中, 会先判断 earlySingletonObjects, 不存在才判断 singletonFactories, 而从 singletonFactories 中取得对象后, 则会将其从 singletonFactories 移除并加入 earlySingletonObjects

这是因为 singletonFactories 缓存的 FactoryBean, 若反复调用 getObject(), 则每次获取都会调用 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#getEarlyBeanReference 方法, 而此方法会执行 SmartInstantiationAwareBeanPostProcessor 的 getEarlyBeanReference(), 这会导致 BeanPostProcessor 重复执行, 显然是不行的.
```



### Spring可以用注解替换XML配置文件了, 是如何实现的呢(常用注解的实现原理)?

```
1) 首先是指定包名或指定类名
	如指定包名则 scan 时会执行, 如指定类名则在构造方法初始化 reader 时执行
2) 无论哪种, 最终都会走一段代码 AnnotationConfigUtils#registerAnnotationConfigProcessors()
3) 这段代码会添加一些 BeanFactoryPostProcessor
	如 ConfigurationClassPostProcessor 负责解析 @Configuration/@Import/@Bean 等注解
    	然后由 ConfigurationClassBeanDefinitionReader 负责将信息转换成BeanDefinition再注册到容器。
	如 AutowiredAnnotationBeanPostProcessor 负责解析 @Autowired/@Value 注解
    如 CommonAnnotationBeanPostProcessor 负责解析 @Resource 注解
    解析放在 postProcessProperties() 方法中， 先扫描bean的字段和方法， 然后一一调用方法和为字段注入值
4) 之后, 他会将扫描的类放到 beanDefinitions 中(或指定的类注册进去)
5) BeanFactory加载完毕后, 回到AbstractApplicationContext的refresh逻辑
	如会执行 postProcessBeanFactory(), 调用前面加入的ConfigurationClassPostProcessor
	然后会添加更多的类到容器中.
    
注意事项：
    @Configuration 和 @Component的区别？
    观察发现，即使使用@Component 其下带 @Bean 的方法依然可以注入到容器中。所以似乎两者没有区别？
    仔细查看源码和资料后，发现 postProcessBeanFactory() 方法在 processConfigBeanDefinitions() 后还会调用 enhanceConfigurationClasses()
    而在这个方法中, 对前面解析了class 是 CONFIGURATION_CLASS_FULL (即代表@Configuration)的类
    会生成一个 cglib 的代理, 这样获取@Bean注解的方法的bean时,不会每次调用方法new 一个, 而是有缓存.
```

> 总结: 就是利用 BeanFactoryPostProcessor 可获取 BeanDefinitionRegistry 对象, 然后扫描容器内带有注解的 bean, 解析这些注解得到一些 BeanDefinition, 再通过获得的 BeanDefinitionRegistry对象注册到 BeanFactory 中.

### Spring AOP是如何实现的(指@Aspect)?

```
1) 使用 @EnableAspectJAutoProxy
2) @EnableAspectJAutoProxy 中使用了 @Import(AspectJAutoProxyRegistrar.class)
3) ConfigurationClassPostProcessor 会解析@Import, 进入 registerBeanDefinitions() 中
4) registerBeanDefinitions() 中添加了 AnnotationAwareAspectJAutoProxyCreator 到容器中
5) AnnotationAwareAspectJAutoProxyCreator 本质上时一个 BeanPostProcessor
6) 因此在 createBean 时, 会被自动调用. 其中 postProcessAfterInitialization() 负责创建代理对象
7) 而 getAdvicesAndAdvisorsForBean() 则负责查找对应的增强. 然后会调用子类的findCandidateAdvisors
8) 如 AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors() 负责注解编写增强@Before/@After等
9) 简单说下逻辑, 就是查找容器所有类, 判断这个类有没有 @Aspect 注解, 然后先找出所有Pointcut
	再遍历所有方法, 找出方法上带有@Before等注解且有关联的Pointcut的方法,
    然后使用这个方法和关联的Pointcut 来new 一个Advisor, 加入到Advisor集合中, 遍历结束后返回即可.
10) 查找到所有的增强后, 再比较Pointcut表达式是否匹配当前的bean, 如可以则加入.
11) 根据找到的Advisor集合, 创建一个带配置(advisor集合等)的代理对象, 代理对象执行方法前
12) 会先根据配置中的advisor集合生成一个执行链, 然后在拦截代理方法处调用. 执行链会负责执行通知.
13) 不同的通知由不同的适配器执行.
```

> 总结就是通过 @EnableAspectJAutoProxy 的@Import, 使得程序最终会执行 AnnotationAwareAspectJAutoProxyCreator 的 postProcessAfterInitialization(对象初始化后调用) 方法, 这个方法在 BeanFactory创建完对象后触发, 此时便可通过 CGlib 等动态代理技术为 创建的 bean 对象创建一个代理对象, 然后这个代理对象会根据 Pointcut 找到关联的 Advisor,  并在合适的时机执行对应的 Advisor, 如 @Before产生的Advisor 会在执行了 bean 对象的指定方法(看Pointcut配置)后执行.

### Spring 事务是如何实现的(指@Transaction)?

```
0) 事务是由AOP实现的, 所以需要找到对应的Pointcut 和 Advisor
1) 打开了 @EnableTransactionManagement 注解
2) 然后@Import 了 TransactionManagementConfigurationSelector
3) 之后导入了 ProxyTransactionManagementConfiguration 到容器中
4) ProxyTransactionManagementConfiguration 带有 @Configuration
5) @Bean 注入了一个通用的Advisor: BeanFactoryTransactionAttributeSourceAdvisor
6) 这个Advisor的 Pointcut 是由 TransactionAttributeSourcePointcut 实现的
	实现逻辑是 TransactionAttributeSourcePointcut 的 matches()
    这个方法调用了 getTransactionAttributeSource() 获取 AnnotationTransactionAttributeSource
    然后通过 getTransactionAttribute() 调用了 findTransactionAttribute()
    最终使用SpringTransactionAnnotationParser 类判断方法是否有@Transactional注解
    并解析注解信息然后返回. 另外这个方法还可以获取@Transactional注解的信息, 而这里只用于判断是否需要拦截这个方法.
7) TransactionInterceptor 是一个Advisor
    也可以通过AnnotationTransactionAttributeSource获取@Transactional注解上的信息
    然后在invoke中, 拦截方法, 打开事务, 在执行完方法后, 提交事务, 报错时回滚事务
    这个 Advisor 不同于传统的前置/后置, 而是更具体的 MethodInterceptor(动态代理直接相关).
```

> 总结: 就是基于AOP实现的, 只需找到对应的 Pointcut 和 Advisor 即可. Pointcut 就是根据 @Transaction 注解判断方法是否需要代理, 这个很简单; 比较有意思的是 Advisor 不是我们写AOP那种 @Before,@Around之类的, 而是更接近动态代理原始的语法的 MethodInterceptor 即 TransactionInterceptor.



## BeanFactoryPostProcessor 相关类分析

### BeanFactoryPostProcessor 生效原理

> 生效原理就是, ApplicationContext 的 refresh 方法中会扫描出容器中实现了 BeanFactoryPostProcessor 接口的 bean, 将其排序后执行相应的接口, 这样我们写的类实现的相应的接口的方法就被执行了.

```bash
常用的 BeanFactoryPostProcessor
# ConfigurationClassPostProcessor
这个类作用就是解析 @Configuration/@Component/@Import/@ImportSource/@ComponentScan 等基础注解. 是注解开发的基石, 更是 Spring Boot 的基石.
```



## BeanPostProcessor 相关类分析

### BeanPostProcessor 生效原理

> 在 refresh() 中会扫描容器中所有 实现了 BeanPostProcessor 接口的类, 添加到 BeanFactory 的 beanPostProcessors 字段中(是个List[CopyOnWriteArrayList自定义版, 自定义加入了清空缓存的逻辑]), 然后在 BeanFactory 创建对象时 createBean() 在适当的时机调用对应的方法.

```bash
有哪几种 BeanPostProcessor (默认的+扩展)
1.InstantiationAwareBeanPostProcessor
	postProcessAfterInstantiation: 对象实例化后调用
	postProcessBeforeInstantiation: 对象实例化前调用
	postProcessProperties: 设置属性值前
	postProcessPropertyValues: 设置属性值前, 若上个方法不处理(返回null)才会触发

2.SmartInstantiationAwareBeanPostProcessor
  predictBeanType: 获取一个 bean 的 class 类型前调用
  getEarlyBeanReference: 获取一个二级缓存对象(singletonFactories的getObject)时调用
  determineCandidateConstructors: 决定一个 bean 实例化的构造参数是什么时调用
	
3.DestructionAwareBeanPostProcessor
	postProcessBeforeDestruction: 对象销毁前调用
	requiresDestruction: 判断这个类针对某个 bean 是否执行 postProcessBeforeDestruction()
	
4.MergedBeanDefinitionPostProcessor
  postProcessMergedBeanDefinition: 在创建对象前调用, 可对 BeanDefinition 做修改
  resetBeanDefinition: 在重置 BeanDefinition 时调用, 用于清空 PostProcessor 对应的缓存
	
5.BeanPostProcessor(基础)
  postProcessBeforeInitialization: 创建对象后(也设置好了字段), 在调用 init 之前调用
  postProcessAfterInitialization: 在创建对象时, 调用了 init 之后调用
  
总结: 
0.对 BeanDefinition 做干预
1.对象实例化过程中(对class/构造参数进行干预)
2.对象实例化前后
3.对象设置属性前, 对属性做干预
4.对象初始化(init)前后
5.对象销毁前

```

### 调用时机

```java
    // 1.1: InstantiationAwareBeanPostProcessor 的 postProcessAfterInstantiation()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean 第一段
		// 1.2: InstantiationAwareBeanPostProcessor 的 postProcessProperties()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean 第二段
		// 1.3: InstantiationAwareBeanPostProcessor 的 postProcessPropertyValues
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean 第三段
		// 1.4: InstantiationAwareBeanPostProcessor 的 postProcessBeforeInstantiation()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation 中

		// 2.1: SmartInstantiationAwareBeanPostProcessor 的 predictBeanType()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.predictBeanType 中
		// 2.2: SmartInstantiationAwareBeanPostProcessor 的 getEarlyBeanReference()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getEarlyBeanReference 中
		// 2.3: SmartInstantiationAwareBeanPostProcessor 的 determineCandidateConstructors()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.determineConstructorsFromBeanPostProcessors 中

		// 3.1: MergedBeanDefinitionPostProcessor 的 postProcessMergedBeanDefinition()
		//   在 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyMergedBeanDefinitionPostProcessors 中
		// 3.2: MergedBeanDefinitionPostProcessor 的 resetBeanDefinition()
		//   在 org.springframework.beans.factory.support.DefaultListableBeanFactory.resetBeanDefinition 中

		// 4.1: DestructionAwareBeanPostProcessor 的 postProcessBeforeDestruction()
		//   在 org.springframework.beans.factory.support.DisposableBeanAdapter.destroy 中
		// 4.2: DestructionAwareBeanPostProcessor 的 requiresDestruction()
		//   在 org.springframework.beans.factory.support.DisposableBeanAdapter.filterPostProcessors 和 org.springframework.beans.factory.support.DisposableBeanAdapter.hasApplicableProcessors 中


```

> 😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁😁

```bash
# 有哪些常用的 BeanPostProcessor
1.AsyncAnnotationBeanPostProcessor: 用于在将 @Async 相应的 Advisor 加入到对象的代理中
2.ScheduledAnnotationBeanPostProcessor: 用于处理 @Scheduled 注解, 将 bean 生产代理类
3.AnnotationAwareAspectJAutoProxyCreator: AOP 实现核心类
4.AutowiredAnnotationBeanPostProcessor: 用于处理 @Autowired 注解
5.ApplicationListenerDetector: 用于处理实现 ApplicationListener 接口的 bean 对象, 将其添加到事件广播器的监听者集合中.
...
```

