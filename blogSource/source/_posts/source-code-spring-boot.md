---
title: Spring Boot 源码笔记
date: 2021-01-24 02:04:22
tags: 
  - spring-boot
categories:
  - [java]
  - [spring-boot]
  - [source-code]

---

# Spring Boot 源码分析



## run 流程

```bash
1.StopWatch 提供的计算耗时的功能, 创建一个后立即开始计时.
2.创建一个引导容器, 并在此时(容器未使用前)把 spring.factories 找到 Bootstrapper 接口的类对应的方法触发, 来给引导容器里注册一些东西(如果有需要)
3.从 spring.factories 找 SpringApplicationRunListener 的类, 实例化后存到 SpringApplicationRunListeners 中.
4.触发所有存入的 SpringApplicationRunListener 的 starting 事件.
5.将 args 内容中的参数们(类似 --spring.port=9999)解析成键值对存到 applicationArguments 对象中.
6.创建了一个 environment 对象, 添加了好几个功能各异的 PropertySource, 触发所有存入的 SpringApplicationRunListener 的 environmentPrepared 事件
7.根据 this.bannerMode 判断是否打印 Banner, 以及打印在哪里, 根据 this.banner 判断打印什么样的 Banner (佛祖保佑.png)
8.根据 web 类型创建不同的 ApplicationContext, 这里的创建, 仅实例化而已, 没有从构造方法调用 loadBeanDefinitions 和 refresh 的逻辑
9.将 applicationStartup (步骤记录器)赋值给 context.
10.对容器做些配置, 然后发布 contextPrepared 事件, 接着关闭引导容器; 然后使用 BeanDefinitionLoader 扫描解析 getAllSources (如Class) 并将得到的 BeanDefinition 注册到容器, 最后发布 contextLoaded 事件
11.注册一个钩子, 当 JVM 关闭时, 相应的关闭 context, 然后调用容器的 refresh 方法(然后进入到 Spring 源码分析那段, 自行脑补)
12.StopWatch 计时器停止计时, 接着打印计时数据
13.发布 started 事件
14.从容器中取出 ApplicationRunner/CommandLineRunner 两类 bean, 并调用它们的 run 方法.
15.catch 到异常则发布 failed 事件
16.发布 running 事件
```

> 从整体结构下来, 我们发现, 其主要是创建了一个引导容器(似乎也就是事件监听用到了, 其他地方完全无瓜), 然后扫描得到一些 SpringApplicationRunListener 存起来, 之后在合适的地方发布事件, 然后创建以及配置 environment 对象, 创建以及配置 ApplicationContext 对象, 解析 primarySources 来加载 BeanDefinition 到容器, 然后调用容器的 refresh 方法进入 Spring 的加载流程, 最后处理事件和调用 ApplicationRunner/CommandLineRunner 的 bean. 其中 加载 BeanDefinition 和执行  refresh 方法其实就是与 Spring 一样的逻辑.



### SpringApplication 与 ApplicationContext 的关系与联系

```bash
1.SpringApplication 有独特的 environment 对象, 这是因为 application.xml 配置以及 Spring Cloud Config 这些都需要在容器创建前加载配置文件.
2.ApplicationContext 是 new 的时候就会立刻触发 加载 BeanDefinition 和 refresh(), 而 SpringApplication 则是先 new 一个, 配置好后, 才分别调用方法去 加载 BeanDefinition 和 refresh().


总得来讲, SpringApplication 相比 ApplicationContext 多出的就是 SpringApplicationRunListener 的事件管理, environment 对象加载与配置, 以及 run 方法的生命周期.
```



### 这些 SpringApplicationRunListener 事件都被谁监听了, 有什么作用?

```bash
# 在 spring-boot-project/spring-boot/src/main/resources/META-INF/spring.factories
#   有一个 SpringApplicationRunListener, 其作用是:
org.springframework.boot.context.event.EventPublishingRunListener
1.构造方法: 初始化一个事件广播器, 并将 SpringApplication 的 listeners 注册进去. (listeners 是在构造方法中从 spring.factories 加载 ApplicationListener 得到的数据)
2.转发 starting,environmentPrepared,contextPrepared,contextLoaded 事件给 ApplicationListener
3.在 contextLoaded 事件时, 遍历所有的 ApplicationListener 对象, 若其实现了 ApplicationContextAware 接口, 则将 context 注入.
4.started,running,failed 事件均直接用 context.publishEvent 发布事件, 与 listeners 无关 (而且 listeners 这些东西如果仅配置在 spring.factories 而没有被扫描到容器内, 那么就是真的无关了)

# 总结: 配置在 spring.factories 的 ApplicationListener 并不会触发所有 run 生命周期的事件. 因此有时实现 SpringApplicationRunListener 还是很有必要的) 当然, 若你写的 ApplicationListener 即配置在 spring.factories 中也会被扫描到容器内, 则无此忧虑.

# 接着看 ApplicationListener 的作用
1.ClearCachesApplicationListener: 在 ContextRefreshedEvent(容器加载完后) 清空缓存
2.ParentContextCloserApplicationListener: 给子容器加一个监听, 使得父容器关闭后, 子容器也跟着关闭(还会递归子容器的容器吧.png)
3.FileEncodingApplicationListener: 对比配置的编码格式, 不符合则报异常(若配置了)
4.DelegatingApplicationListener: 新建一个事件广播器, 将配置文件中 context.listener.classes 的 class 实例化并作为监听者注册到广播器, 然后转发所有事件.
5.EnvironmentPostProcessorApplicationListener
    接受 ApplicationEnvironmentPreparedEvent 事件, 然后把从 spring.factories 获得的 EnvironmentPostProcessor 的 classNames 实例化, 然后遍历执行 postProcessEnvironment(), 其中 ConfigFileApplicationListener 用于加载 application.xxx(yml,xml,properties) 文件的配置到 environment 中的 PropertySource 里. (另外一提, Spring Cloud Config 应该也是这里实现的)

# 总结: 原来配置文件是这么被加载进去的, 没有在主流程中写, 而是监听事件, 再调用 EnvironmentPostProcessor, 层层封装, 扩展性好强(读起来也好蓝).
```



### 超长源码注释

```java
// 我们从源码的 spring-boot/spring-boot-tests/spring-boot-smoke-tests/spring-boot-smoke-test-tomcat 下, 找到 SampleTomcatApplication.java 类, 直接看它的 main 方法.

public static void main(String[] args) {
  SpringApplication.run(SampleTomcatApplication.class, args);
}
```

接着, 我们点进 run 方法, 瞧瞧里面干了啥

```java
// 看起来不多嘛......(看前如是写道, 看完已是凌晨四点)
public ConfigurableApplicationContext run(String... args) {
  // 1.StopWatch 提供的计算耗时的功能, 创建一个后立即开始计时.
  // 2.创建一个引导容器, 并在此时(容器未使用前)把 spring.factories 找到 Bootstrapper 接口的类对应的方法触发, 来给引导容器里注册一些东西(如果有需要)
  // 3.从 spring.factories 找 SpringApplicationRunListener 的类, 实例化后存到 SpringApplicationRunListeners 中.
  // 4.触发所有存入的 SpringApplicationRunListener 的 starting 事件.
  // 5.将 args 内容中的参数们(类似 --spring.port=9999)解析成键值对存到 applicationArguments 对象中.
  // 6.创建了一个 environment 对象, 添加了好几个功能各异的 PropertySource, 触发所有存入的 SpringApplicationRunListener 的 environmentPrepared 事件
  // 7.根据 this.bannerMode 判断是否打印 Banner, 以及打印在哪里, 根据 this.banner 判断打印什么样的 Banner (佛祖保佑.png)
  // 8.根据 web 类型创建不同的 ApplicationContext, 这里的创建, 仅实例化而已, 没有从构造方法调用 loadBeanDefinitions 和 refresh 的逻辑
  // 9.将 applicationStartup (步骤记录器)赋值给 context.
  //10.对容器做些配置, 然后发布 contextPrepared 事件, 接着关闭引导容器; 然后根据 primarySource 使用 BeanDefinitionLoader 加载 BeanDefinition 到容器, 最后发布 contextLoaded 事件
  //11.注册一个钩子, 当 JVM 关闭时, 相应的关闭 context, 然后调用容器的 refresh 方法(然后进入到 Spring 源码分析那段, 自行脑补)
  //12.计时器停止计时, 接着打印计时数据
  //13.发布 started 事件
  //14.从容器中取出 ApplicationRunner/CommandLineRunner 两类 bean, 并调用它们的 run 方法.
  //15.catch 到异常则发布 failed 事件
  //16.发布 running 事件

  StopWatch stopWatch = new StopWatch();
  stopWatch.start();

  // 创建一个引导容器, 并在此时(容器未使用前)从 spring.factories 扫描一些实现了 Bootstrapper 接口的类, 来给引导容器里注册一些东西(如果有需要)
  DefaultBootstrapContext bootstrapContext = createBootstrapContext();
  ConfigurableApplicationContext context = null;

  // 往系统变量里设置一个变量 headless, 看上去和 AWT 有关, 暂且忽略之
  configureHeadlessProperty();

  // 扫描 spring.factories 中配置的实现了 SpringApplicationRunListener 的类们
  //   实例化后放到 SpringApplicationRunListeners (其就是个容器管理类) 存起来.
  SpringApplicationRunListeners listeners = getRunListeners(args);

  // 触发所有存入的 SpringApplicationRunListener 的 starting 事件.
  listeners.starting(bootstrapContext, this.mainApplicationClass);
  try {
    // 将 args 内容中的参数(类似 --spring.port=9999)解析成键值对存到 applicationArguments 对象中.
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

    // 创建了一个 environment 对象, 添加了好几个功能各异的 PropertySource,
    //   触发所有存入的 SpringApplicationRunListener 的 environmentPrepared 事件
    //   将 environment 中 spring.main 开头的配置数据, 一一对应绑定到 SpringApplication(即this)的字段上去
    ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);

    // 将 environment 的 spring.beaninfo.ignore 配置复制到 System.Property 去(若不存在)
    configureIgnoreBeanInfo(environment);

    // 根据 this.bannerMode 判断是否打印 Banner, 以及打印在哪里, 根据 this.banner 判断打印什么样的 Banner (佛祖保佑.png)
    Banner printedBanner = printBanner(environment);

    // 根据 web 类型创建不同的 ApplicationContext, 但其实差别不大, 就时多了 web 容器的特征(如启动 tomcat)
    // 另外与 Spring 源码分析时不同, 这里的创建, 仅实例化而已, 没有从构造方法调用 loadBeanDefinitions 和 refresh 的逻辑
    context = createApplicationContext();

    // 将 applicationStartup (步骤记录器)赋值给 context.
    context.setApplicationStartup(this.applicationStartup);

    // 对 context 做一些配置, 执行 initializers 的 initialize()
    // 发布 contextPrepared 事件
    // 关闭引导容器, 即发布 BootstrapContextClosedEvent 事件给之前加的 closeListener (监听者)
    // 使用 BeanDefinitionLoader 根据 sources 和 run 方法参数 primarySource 加载 BeanDefinition 到容器中.
    // 发布 contextLoaded 事件
    prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);

    // 注册一个钩子, 当 JVM 关闭时, 相应的关闭 context, 然后调用容器的 refresh 方法(然后进入到 Spring 源码分析那段, 自行脑补)
    refreshContext(context);

    // 留给子类扩展吧
    afterRefresh(context, applicationArguments);

    // 计时器停止计时, 接着打印计时数据
    stopWatch.stop();
    if (this.logStartupInfo) {
      new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
    }

    // 发布 started 事件
    listeners.started(context);

    // 从容器中取出 ApplicationRunner/CommandLineRunner 两类 bean, 并调用它们的 run 方法.
    callRunners(context, applicationArguments);
  }
  catch (Throwable ex) {
    // 发布 failed 事件
    handleRunFailure(context, ex, listeners);
    throw new IllegalStateException(ex);
  }

  try {
    // 发布 running 事件
    listeners.running(context);
  }
  catch (Throwable ex) {
    // 发布 failed 事件
    handleRunFailure(context, ex, null);
    throw new IllegalStateException(ex);
  }
  return context;
}
```

> 温馨提示: 先在想看的地方打断点, 在打开调试模式, 可以清楚的看到对应的变量变化情况, 以此理解代码!

```java
// 来看看所有过程涉及方法的注释(大多数)
org.springframework.boot.SpringApplication#createBootstrapContext
    // 1.创建一个引导容器(这个容器作用和 BeanFactory 类似, 但更简单得多, 仅有获取/注册 bean 对象等的功能)
		//   触发存于 this.bootstrappers 的对象的 intitialize 方法来对引导容器进行初始化(即可以在引导容器未使用前往里面注册 bean 对象)
		//   然后 this.bootstrappers 的数据有一部分是从 spring.factories 找 Bootstrapper 的类实例化后得到的, 然后也可以代码手动添加
		
  
org.springframework.boot.SpringApplication#getRunListeners
    // 扫描 spring.factories 中配置的实现了 SpringApplicationRunListener 的类, 并调用形如 types 的参数的构造函数来实例化得到对象集合
		//   再将这些对象集合放到 SpringApplicationRunListeners(其就是个容器管理类)中.
		// 并绑定 applicationStartup (步骤记录器)
  

org.springframework.boot.SpringApplicationRunListeners#doWithListeners(java.lang.String, java.util.function.Consumer<org.springframework.boot.SpringApplicationRunListener>, java.util.function.Consumer<org.springframework.core.metrics.StartupStep>)
    // 遍历持有的所有 SpringApplicationRunListener, 触发对应事件
		// 在触发监听事件前后加入 StartupStep 监听所耗时间
		// 步骤监听器的 accept 处理
  
  
org.springframework.boot.SpringApplication#prepareEnvironment
    // 1.根据 web 类型创建一个 environment 对象
		// 2.配置 environment
		//     通过 ConversionService 类添加一些类型转换支持(如文件大小1024k=1M等)
		//     将用户设置的 defaultProperties 添加到 environment 的 sources 中, 将 args 解析成一个 PropertySource 后加入到 environment 中
		// 3.为 environment 添加一个名为 configurationProperties 的源, 其作用是将每一个 PropertySource 适配成 ConfigurationPropertySource.
		// 4.触发所有存入的 SpringApplicationRunListener 的 environmentPrepared 事件.
		// 5.将名为 defaultProperties 的 Property 源移最后(降低优先级), 另此 defaultProperties 不是 this.defaultProperties
		// 6.将用户通过代码设置的要附加的 profile 设置到 activeProfiles 中去 (若存在且 environment 中不存在)
		// 7.将 environment 中 spring.main 开头的配置数据, 一一对应绑定到 SpringApplication(即this)的字段上去
		// 8.若不开启自定义 environment, 则将 environment 转换成 StandardEnvironment(默认行为)
		// 9.由于上一步可能做了转换, 所以需要重新 attach 一次.

  
org.springframework.boot.SpringApplication#getOrCreateEnvironment
    // 1.若存在一个 environment, 则直接返回
		// 2.若不存在, 则根据之前推断得出的 webApplicationType 来创建对应的 Environment 对象.
		// 3.这几个不同的类型区别也不大, 就 StandardServletEnvironment 多了3个 propertySources(其中一个是 JDNI, 比较重要)
		
  
org.springframework.boot.SpringApplication#configureEnvironment
    // 1.通过 ConversionService 类添加一些类型转换支持(如文件大小1024k=1M等)
		// 2.将用户设置的 defaultProperties 添加到 environment 的 sources 中, 将 args 解析成一个 PropertySource 后加入到 environment 中.
		// 3. configureProfiles 是一个空方法, 看来是留给我们实现子类时扩展的.
		
  
org.springframework.boot.context.properties.source.ConfigurationPropertySources#attach
    // 为 environment 添加一个名为 configurationProperties 的源, 其作用是将每一个 PropertySource 适配成 ConfigurationPropertySource.
		
  
org.springframework.boot.SpringApplication#configureAdditionalProfiles
    // 将用户通过代码设置的要附加的 profile 设置到 activeProfiles 中去 (若存在且 environment 中不存在)
		
  
org.springframework.boot.SpringApplication#bindToSpringApplication
    // 将 environment 中的配置数据, 绑定到 SpringApplication(即this)的一些字段上去
		// 绑定规则是 spring.main 开头的配置数据与 SpringApplication 一一对应, 如若存在 spring.main.banner-mode=OFF, 则 this.bannerMode=OFF
		
  

  
org.springframework.boot.SpringApplication#printBanner
    // 根据 this.bannerMode 判断是否打印 Banner, 以及打印在哪里
		//   this.banner 为文件路径, 若为空, 则打印 Spring Boot 默认准备的文本(这不换个佛祖保佑?)
		
  
org.springframework.boot.SpringApplication#createApplicationContext
  
    // 根据 web 类型创建不同的 ApplicationContext, 但其实差别不大, 就几个细节不同罢了
		// 以 AnnotationConfigServletWebServerApplicationContext 为例, 与 AnnotationConfigApplicationContext 的区别大致为
		//   多加了一个 BeanPostProcessor 用于给 ServletContextAware/ServletConfigAware 接口注入 servletContext/servletConfig 对象
		//   onRefresh() 时, 调用 createWebServer() 启动 tomcat/jetty/undertow
		
  
org.springframework.boot.SpringApplication#prepareContext
    // 1.设置 environment 对象
		// 2.将 beanNameGenerator 注册到容器中 (若存在), 配置容器的 resourceLoader 和 conversionService (从 SpringApplication 获取)
		// 3.将 initializers 根据 @Order 配置排序后, 遍历执行其 initialize 方法.
		// 4.发布 contextPrepared 事件
		// 5.关闭销毁引导容器(毕竟真正的容器已经准备好了, 这玩意就没用了)
		// 6.为容器添加一些特殊的 bean, 对 beanFactory 做点小设置; 然后添加一个懒加载功能的 BeanFactoryPostProcessor, 作用是将所有的 BeanDefinition 的 lazyInit 设置为 true
		// 7.创建一个 BeanDefinitionLoader, 解析 sources 得到 BeanDefinition 再注册到容器中. 和 Spring 的 new 容器时的 load 过程类似.
		// 8.发布 contextLoaded 事件

  
org.springframework.boot.SpringApplication#postProcessApplicationContext
    // 1.将 beanNameGenerator 注册到容器中 (若存在)
		// 2.将 SpringApplication 的 resourceLoader 赋值给容器 (若存在)
		// 3.为容器设置 ConversionService(类型转换工具)对象 (若配置了允许)
		
  
org.springframework.boot.SpringApplication#load
    // 创建一个 BeanDefinitionLoader, 解析 sources 得到 BeanDefinition 再注册到容器中. 和 Spring 的 new 容器时的 load 过程类似.
		

org.springframework.boot.SpringApplication#refreshContext
    // 1.注册一个钩子, 当 JVM 关闭时, 相应的关闭 context
		// 2.调用容器的 refresh 方法
		
  
org.springframework.boot.SpringApplication#callRunners
    // 从容器中取出 ApplicationRunner/CommandLineRunner 两类 bean, 并调用它们的 run 方法.
		// 这里有两个不同点
		//    1. XxxRunner 和 ApplicationListener 有何不同?
		//       答案是 XxxRunner 的 run 方法可以直接取到程序启动的 args 参数, 而监听器要取则还需借助 environment
		//    2.ApplicationRunner 和 CommandLineRunner 有何不同?
		//       答案是 run 方法接受的参数形式不同, 一个是 字符串数组(原始的), 一个是解析好的 key:value 方便直接取用.
		
  

```

> 规矩我懂得, GitHub在这 [注意分支哦](https://github.com/gudqs7/spring-boot/tree/wq-comment) 



## 各种实现的原理

```bash
1.application.properties 是如何被加载到 Environment 中的?
2.@ConfigurationProperties 如何实现自动注入`application.properties/application.yml`中配置的值?
3.一些只需要改改依赖jar就可以切换(如tomcat->undertow)是怎么做到的? (@ConditionalXxx 的实现原理)
4.JdbcTemplateAutoConfiguration如何确保能够获取到 DataSource? (@AutoConfigureAfter 的实现原理)
5.为什么 Spring Boot 启动 main 方法就能访问 Tomcat? (SpringApplication.run() 启动 tomcat 实现原理)
```



### application.properties 是如何被加载到 Environment 中的?

```bash
1) run方法中创建了 Environment 对象, 当初始化好一些东西后会触发 environmentPrepared 事件
2) 通过 SpringApplicationRunListener 的 EventPublishingRunListener 转发事件给 ApplicationListener 这类监听者, 即 EnvironmentPostProcessorApplicationListener.
3) 这个类接收事件后, 遍历从 spring.factories 获得的 EnvironmentPostProcessor 对象, 执行其 postProcessEnvironment()
4) 其中 ConfigFileApplicationListener#postProcessEnvironment() 实现了配置文件的加载
5) 具体为 postProcessEnvironment 下的 addPropertySources()
6) 此方法将会扫描指定的路径下指定的某些文件
7) 然后使用 spring.factories 下的 PropertySourceLoader 一一尝试解析
8) 文件存在且解析正确则加入到 environment 的 sources 集合中.
	某些路径: getSearchLocations() ,默认: classpath:/,classpath:/config/ ...
	某些文件: getSearchNames() ,默认: application

# PS: 
PropertySourceLoader 有 PropertiesPropertySourceLoader/YamlPropertySourceLoader
一个尝试后缀有 xml/properties, 另一个是 yml/yaml, 因为是遍历后 load, 所以所有可能性有:
    classpath:/application.xml; classpath:/application.properties
	classpath:/application.yml; classpath:/application.yaml
    ......
```



### @ConfigurationProperties 如何实现自动注入`application.properties/application.yml`中配置的值?

```bash
1) 首先加了 @EnableConfigurationProperties 也会解析里面的 @Import 
2) @Import 则引入了 EnableConfigurationPropertiesRegistrar.class
3) 这是一个 ImportBeanDefinitionRegistrar 的实现类, 因此调用指定方法 registerBeanDefinitions
4) 指定方法 registerBeanDefinitions() 会将 @EnableConfigurationProperties 注解的值对应的类注册到容器中, 如 @EnableConfigurationProperties(RabbitProperties.class) 则会加载 RabbitProperties.class
5) 指定方法还注册了一些工具 bean 和一个重要的 BeanPostProcessor 在 registerInfrastructureBeans()中
6) 即 ConfigurationPropertiesBindingPostProcessor, 当我们要使用配置文件 bean(如 RabbitProperties)时,会实例化 bean 并触发 postProcessorBeforeInitialization()
7) 在 postProcessorBeforeInitialization() 中, 通过 org.springframework.boot.context.properties.ConfigurationPropertiesBinder#bind() 来完成实际的绑定
8) 其本质就是 Binder 的 bind 方法, 设定配置文件前缀即可将配置文件中的配置对应的绑定到 bean(如 rabbitProperties 对象) 中.

# PS
这里的 Binder 和 SpringApplication 里绑定 spring.main 开头配置文件那个类是同一个.

# 总结
就是先将 XxxProperties 类注册到容器中, 这样就可以通过 BeanPostProcessor 再实例化后将配置文件与属性绑定.
```



### 一些只需要改改依赖jar就可以切换(如tomcat-->undertow)是怎么做到的? (@ConditionalXxx 的实现原理)

```bash
1) 在类上加上注解 @Conditional 或带有 @Conditional 的其他注解(即注解的子类亦会扫描)
2) 在所有的扫描类和注解的地方,如解析@Configuration, AnotatedBeanDefinitionReader 等 reader
    会使用 ConditionEvaluator 的 shouldSkip() 判断是否可以加载, 时机点如下
    AnnotatedBeanDefinitionReader#doRegisterBean() 的第二行代码
    ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForBeanMethod() 第四行
    ConfigurationClassBeanDefinitionReader#loadBeanDefinitionsForConfigurationClass()
    ConfigurationClassParser#doProcessConfigurationClass() 处理 ComponentScan 那段
3) 然后再 shouldSkip 中判断， 判断逻辑大致如下：
    先遍历所有注解取得所有的 @Conditional 的 value, 这个 value 是具体的 Condition 实现, 如 OnClassCondition(实现了 Condition 的 matches 接口, 很多抽象类用以增强代码扩展性), 然后其加入到 conditions 中, 排序后遍历调用 matches(), 一个不匹配则返回 true, 代表应该跳过.

# TIPS:
ConditionOutcome 封装了是否匹配和匹配日志信息[为啥成功/为啥失败]
SpringBootCondition 提供了通用的根据 ConditionOutcome 判断是否匹配并记录日志信息的抽象类.
    子类只需实现 getMatchOutcome(): 根据 metadata[注解信息] 返回 ConditionOutcome 对象.
    因此, 如果我们要实现自己的 Condition, 可以继承它.
```



### JdbcTemplateAutoConfiguration 如何确保能够获取到 DataSource? (@AutoConfigureAfter 的实现原理)

```bash
1) 首先所有的 XxxAutoConfiguration 都是借助 AutoConfigurationImportSelector 加载的, 其继承了 DeferredImportSelector.  
2) 这种 DeferredImportSelector 会延迟加载, 原理是 parse 后再加载, 而非 parse 执行过程中就加载.
3) 延迟加载机制 会先调用 process 方法, 将要加载的 class 保存起来, 然后再调用 selectImports 返回.
4) 此时 AutoConfigurationImportSelector.AutoConfigurationGroup 的 selectImports() 会调用 sortAutoConfigurations(), 也就是调用了 AutoConfigurationSorter.getInPriorityOrder()
5) getInPriorityOrder() 调用了 sortByAnnotation() 这个方法根据2个注解 @AutoConfigureBefore/@AutoConfigureAfter 排序.
6) 最后返回的就是有序的了. 另外, 这两个注解只能作用在 spring.factories 中 EnableAutoConfiguration 的类上才有效.

# 总结:
利用 DeferredImportSelector 的延迟加载, 将所有的 AutoConfiguration 所引入的 class 先存起来不加载, 然后又加入排序的逻辑, 使得真正加载时会根据排序结果依次加载.
```



### 为什么 Spring Boot 启动 main 方法就能访问 Tomcat? (SpringApplication.run() 启动 tomcat 实现原理)

```bash
1) main 方法会调用 SpringApplication 的 run 方法, 里面会创建 applicationContext, 如是 webApplicationType = SERVLET, 则实现类为 AnnotationConfigServletWebServerApplicationContext
    另外一提, webApplicationType 是根据 classpath 下是否有哪些类来推断的.
2) 这个类继承了 ServletWebServerApplicationContext
3) ServletWebServerApplicationContext 实现了 onRefresh()
4) onRefresh() 调用了 createWebServer()
5) createWebServer() 使用 ServletWebServerFactory.getWebServer() 获取 webServer 对象
6) ServletWebServerFactory 的具体实现类由 ServletWebServerFactoryAutoConfiguration 引入
    详细见 ServletWebServerFactoryConfiguration.EmbeddedTomcat.class
7) 引入后调用 getWebServer(), 大概为 new Tomcat(), 绑定配置文件到 Tomcat 的一些属性上, 然后启动它.
8) 至此, run() 启动了 tomcat/jetty/undertow.
 
# TIPS:
Spring 使用工厂模式获取不同的 webServer, 而不同的 webServer 实现类其实是用 XxxAutoConfiguration 来自动注入的(还可以配合 @Conditional 决定何时加载).
这样如果新增一种 webServer, 只需要在写一个 XxxAutoConfiguration 注册一个 webServer 实现类的 bean 即可, 非常灵活(非常 nice).
```





## 常见的 XxxAutoConfiguration

```bash
1.连接池
   DataSourceAutoConfiguration 会 Import DataSourceConfiguration.Hikari.class
2.Mybatis
   MybatisAutoConfiguration: 注册并配置了 SqlSessionFactory/SqlSessionTemplate
3.Spring MVC
   DispatcherServletAutoConfiguration: 配置前端控制器和文件上传处理器
   ServletWebServerFactoryAutoConfiguration: 注册 WebServerFactory 的实现类 bean
4.RocketMQ
   RabbitAutoConfiguration: 注册并配置 RabbitTemplate
5.Redis
   RedisAutoConfiguration: 注册并配置 RedisTemplate/StringRedisTemplate
6.邮件发送
   MailSenderAutoConfiguration: 注册并配置 JavaMailSenderImpl
7.MyBatis-Plus
   MybatisPlusAutoConfiguration: 注册 SqlSessionFactory, 并配置了其 plugins, 使 Mybatis-Plus 生效
8.定时任务
   TaskSchedulingAutoConfiguration: 注册 ThreadPoolTaskScheduler
9.AOP
   AopAutoConfiguration: 通过静态内部类加载了 @EnableAspectJAutoProxy
```



## 我所见到的设计模式

```bash
1.适配器模式: SpringConfigurationPropertySources
    要将 Iterable<PropertySource<?>>
    适配成 Iterable<ConfigurationPropertySource>
    也即是 PropertySource --> ConfigurationPropertySource
  #其实主要工作是: ConfigurationPropertySource#getConfigurationProperty() 里面调用了 PropertySource#getProperty(), 以此完成适配... 很适配器模式, 存一个未适配的对象, 在适配的方法中调用存储的对象的要适配的方法. 
  
2.简单工厂模式: org.springframework.boot.ApplicationContextFactory#create
    根据 webApplicationType 创建不同的 ConfigurableApplicationContext
  
3.观察者模式: 
	监听者: SpringApplicationRunListener
  观察对象: SpringApplication 的生命周期(starting/environmentPrepared/contextPrepared/contextLoaded/started/running/failed)
  管理者: SpringApplicationRunListeners, 负责遍历监听者广播对应事件
```

