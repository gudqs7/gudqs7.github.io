---
title: Spring MVC 源码笔记
date: 2021-01-25 23:12:41
tags:   
  - spring-mvc
categories:
  - [java]
  - [spring]
  - [source-code]
---

# Spring MVC 源码笔记



## 关键类分析

```bash
WebMvcConfigurationSupport
	默认注册了很多东西,如 HandlerMapping 几个实现, HandlerAdaptor 几个实现

HandlerMapping
	添加容器内所有带有 RequestMaping 的类的公开方法到 mappings 中存起来
		(AbstractHandlerMethodMapping#afterPropertiesSet中)
    根据 request 的 uri 查找对应的 HandlerMethod, 步骤概述:
    	把RequestMapping注解内的 path 作为 key 保持到一个 map1
    	其他信息封装成 mapping 作为 key 也保持到另一个 map2
    	根据 uri去 map1 获取 mapping, 再根据 mapping 获取 HandlerMethod
    	封装成 Match 对象, 与其他匹配对象做比较后, 返回 HandlerMethod

HandlerAdapter
	初始化参数解析，返回值解析等
		(RequestMappingHandlerAdapter#afterPropertiesSet)
	根据 handler 确定对应的 HandlerAdapter, 然后 HandlerAdapter 负责执行这个 handler
    如 RequestMappingHandlerAdapter 则负责执行 HandlerMethod
    	简单说就是封装 HandlerMethod, 根据参数值设置参数, 然后调用方法, 再处理返回值封装成 ModelAndView
    另外，这里如果使用了@ResponseBody，会进入 RequestResponseBodyMethodProcessor
    	然后使用 messageConverters（json）写入到响应流
    	最后 mv 也直接返回null, 不需要render了.

ViewResolver
	负责将 ModelAndView 解析成HTML, 如JSP, FreeMarker

HandlerExecutionChian
	管理拦截器和封装Handler, 负责拦截器的实际调用逻辑实现
	
DispatcherServlet
	调度整个HTTP请求响应流程, 调用各个子组件负责执行处理方法, 解析视图, 处理异常等.
```





## SSM项目 Spring 容器和 Spring MVC 容器的创建过程及关系

### Spring 容器的创建

```bash
org.springframework.web.context.ContextLoader#initWebApplicationContext
1.先判断此方法是否重复运行, 若重复则报异常.
2.记录日志, 记录开始时间用于计算启动消耗时间
3.将容器对象存储在本地而非 servletContext, 防止 ServletContext 关闭后无法访问
4.根据配置的类名, 实例化一个容器并赋值给 this.context
5.标准代码, 但其实就是 setParent(null), 前提是不扩展子类咯. 也就是说这个就是顶级容器了
6.根据 web.xml 的配置对容器做相应的配置, 初始化, 将 ServletContext 存入到 environment 对象中; 执行 InitializerClass, 调用容器 refresh 完成容器加载
7.打印容器初始化完成日志和记录耗时

```

> 总结: 监听 ServletContext 创建完毕的事件, 然后创建一个 web 容器, 配置一些东西(id, parent, environment)并初始化(refresh)



### MVC 子容器的创建

```bash
1.首先是 DispatchServlet 本身是一个 Servlet, 因此他有生命周期 init(), 其父类 init() 会将 ServletConfig 的配置(web.xml 中的 initParams)与 servlet 对象绑定, 这样 DispatchServlet 对象的字段就有值了.
2.然后在 DispatchServlet 的父类 FrameworkServlet#initServletBean 中, 会创建一个容器并使其加载.(详细见 FrameworkServlet#initWebApplicationContext() )

```

> 总结: 就是利用 Servlet 的生命周期只会执行一次 init 的特性, 查找父容器, 创建子容器.

```java
org.springframework.web.servlet.HttpServletBean#init
    // 1.把 servletConfig 的 initParameters 都加入 PropertySource, 根据 requiredProperties 判断所需配置项是否齐全
		// 2.将配置项绑定到 Servlet 对象(this) 上(子类的字段也算).
		// 3.调用子类初始化方法
  
org.springframework.web.servlet.FrameworkServlet#initServletBean
    // 0.父类的 init 执行完, 此时 contextConfigLocation 已经有值了
		// 1.记录日志和开始时间
		// 2.创建子容器, 与父容器绑定, 做一些配置, 调用 refresh() 完成加载.
		// 3.打印日志, 打印耗时

  
org.springframework.web.servlet.FrameworkServlet#initWebApplicationContext
    // 1.从 ServletContext 中获取 Spring 容器.(那么谁放进去的呢? 没错, 就是 ContextLoader,监听了 ServletContext 事件)
		// 2.若容器在构造方法处被注入(可能是注解开发 Spring MVC 的方式会触发), 先忽视  -- 来自 web.xml 形式启动分析
		// 3.根据绑定得到的配置的 contextClass 创建一个子容器, 进行一些配置和初始化, 调用 refresh() 完成容器加载
		// 4.防止子容器不支持 refresh, 或子容器不是刚刚创建的, 因此手动触发 onRefresh(), 这个方法会加载一些默认的 bean(用处很大的那种)
		// 5.将容器设置到 ServletContext 中(根据配置, 默认允许)
		// 6.返回创建的子容器对象

  
org.springframework.web.servlet.FrameworkServlet#configureAndRefreshWebApplicationContext
    // 1.根据绑定得到的配置设置容器 id, 若无则生成默认的
		// 2.设置 ServletContext/ServletConfig 对象
		// 3.为子容器添加一个监听只监听子容器刷新事件的监听器, 用于容器加载完毕后调用 FrameworkServlet.onApplicationEvent()
		// 4.将 ServletConfig/ServletContext 加入到 environment 中.
		// 5.执行 web.xml 中配置的 Initializers, 为子容器做的初始化
		// 6.调用子容器的 refresh(), 完成容器的加载

  
org.springframework.web.servlet.DispatcherServlet#initStrategies
    // 加载文件上传处理 bean
		// 加载国际化处理 bean
		// 加载主题切换处理 bean
		// 加载 HandlerMapping bean
		// 加载 HandlerAdapter bean
		// 加载异常处理 bean
		// 加载 HttpServletRequest 转视图名称处理策略 bean
		// 加载视图解析器
		// 加载 FlashMap 管理 bean

		// 每一个加载的逻辑都是类似的, 先从容器中根据 Xxx.class getBean
		// 若无, 则调用 getDefaultStrategy() 从 DispatcherServlet.properties 配置文件中读取
		

```



### DispatchServlet 的创建

```bash
1.Tomcat 会创建配置在 web.xml 的 Servlet
2.接着会触发 init 方法, init 调用了创建子容器的方法后, 还添加了容器加载完毕事件监听来回调 DispatcherServlet#onRefresh
3.DispatcherServlet#onRefresh 会 initStrategies() 加载很多策略接口 bean.
```

> 总结: 和子容器的创建息息相关.





## Dispatch 过程

```bash
1.覆盖 HttpServlet 的 service 方法, 调用 FrameworkServlet#processRequest()
2.此方法中进行一些上下文的准备工作, 以及处理日志, 异常(非 controller 异常), 然后调用 DispatcherServlet#doService()
3.doService() 中对 request 做一些准备工作, 然后调用 DispatcherServlet#doDispatch()
4.doDispatch() 先用 handlerMappings 查找合适的 handler(并加入拦截器链), 再通过 handlerAdapters 得到 handler 的适配器, 在合适的地方触发拦截器; 然后调用适配器的 handle() 得到 ModelAndView
5.得到 ModelAndView 后, 先判断是否捕获到了异常, 是则调用 handlerExceptionResolvers 的 resolveException() 处理异常
6.接着, 调用 viewResolvers 的 resolveViewName, 将 viewName 解析成一个 View 对象
7.调用 View 对象的 render(), 将视图通过 response 响应到前端.
```

> 总结: handlerMappings 找 handler 并包装拦截器链, handlerAdapters 找可执行的 HandlerAdapter, viewResolvers 解析视图, 渲染视图.



### 超长源码注释

```java
// 入口 HttpServlet.service
@Override
protected void service(HttpServletRequest request, HttpServletResponse response)
  throws ServletException, IOException {

  HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
  if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
    processRequest(request, response);
  }
  else {
    super.service(request, response);
  }
}
```

接着进入到 processRequest, 我只写注释了啊

```java
org.springframework.web.servlet.FrameworkServlet#processRequest
    // 1.记录启动时间
		// 2.准备国际化处理上下文
		// 3.准备属性管理上下文
		// 4.获取并缓存一个 asyncManager(异步)
		// 5.初始化两个 ContextHolder
		// 6.处理文件上传, 根据不同策略查找可执行的 controller 方法, 查到后加入拦截器, 再通过适配器来执行 controller 方法, 然后把得到的 ModelAndView 解析成视图对象, 并渲染到前端.
		// 7.重置两个 ContextHolder
		// 8.打印日志, 发布事件

  
org.springframework.web.servlet.DispatcherServlet#doService
    // 1.打印日志, 记录请求信息
		// 2.保存一份请求上下文的快照, 这样嵌套的请求在回归时可以恢复数据
		// 3.将容器中的一些 bean 配置到请求上下文中
		// 4.处理文件上传, 根据不同策略查找可执行的 controller 方法, 查到后加入拦截器, 再通过适配器来执行 controller 方法, 然后把得到的 ModelAndView 解析成视图对象, 并渲染到前端.
		// 5.若需要, 将快照恢复到请求上下文

  
org.springframework.web.servlet.DispatcherServlet#doDispatch
    // 1.准备一些变量
		// 2.判断并处理文件上传请求, 若是, processedRequest 则变为 MultipartHttpServletRequest 类型的对象.
		// 3.遍历之前加载的 handlerMappings, 调用 getHandler 接口获取执行链对象并返回. 找不到则返回 null
		// 4.根据 handler 获取合适的适配器
		// 5.遍历执行拦截器的 preHandle(), 若遇到 renturn false, 则结束 doDispatch
		// 6.执行实际的 controller 的方法得到 ModelAndView 对象.
		// 7.处理默认的 viewName
		// 8.遍历执行拦截器的 postHandle()
		// 9.若有异常则处理异常: 遍历之前加载的异常处理器策略类, 调用 resolveException()
		//10.若无异常则根据需要根据 ModelAndView的对象的视图名配合之前加载的视图解析器获取 View 对象, 再调用 render() 渲染视图.
		//11.最后执行拦截器的 triggerAfterCompletion

  
org.springframework.web.servlet.DispatcherServlet#getHandler
    // 遍历之前加载的 handlerMappings, 调用其 getHandler 接口获取 handler 和拦截器封装成 chain 对象并返回. 找不到则返回 null
  

org.springframework.web.servlet.DispatcherServlet#getHandlerAdapter
    // 遍历之前加载的 handlerAdapters, 调用 supports 判断是否支持 handler, 支持则返回 adapter
  
  
org.springframework.web.servlet.HandlerExecutionChain#applyPreHandle
    // 从头到尾遍历拦截器, 执行 preHandle(), 若拦截器返回 false, 则立刻 triggerAfterCompletion, 然后 return false
  
  
org.springframework.web.servlet.DispatcherServlet#processDispatchResult
    // 若有异常则处理异常: 遍历之前加载的异常处理器策略类, 调用 resolveException()
		// 若无异常则根据需要根据视图名和视图解析器渲染视图
		// 最后执行拦截器的 triggerAfterCompletion

  
org.springframework.web.servlet.DispatcherServlet#processHandlerException
    // 遍历之前加载的异常处理器策略类, 调用 resolveException()
  
  
org.springframework.web.servlet.DispatcherServlet#render
    // 先遍历视图解析器, 根据视图名获取实际视图对象
		// 最后调用实际视图对象的 render (渲染方法)

  
org.springframework.web.servlet.DispatcherServlet#resolveViewName
    // 遍历视图解析器, 根据视图名获取实际视图对象

```





## 某些实现原理

```bash
1.拦截器原理
2.默认的 HandlerMapping/HandlerAdapter 在何时加入?
3.参数是如何与 HTTP 请求 body 绑定的(序列化, 格式化, 绑定)?
4.参数校验是如何进行的?
5.有关参数与返回值的一些拦截与干预(@RequestBody, @ResponseBody).
```



### 拦截器原理

```
何时加入?
	从 WebMvcConfigurationSupport 的子类中调用 addInterceptors 
	添加一些拦截器和拦截器的路径配置 InterceptorRegistry 和 MappedInterceptor
    实现拦截器路径匹配, 在 new HandlerExecutionChian 时判断

何时执行?
	DispatcherServlet 负责在正确的时机调用 HandlerExecutionChian 来调用 preHanlde 等方法.
	拿到 HandlerExecutionChian 后调用 preHanlde
	HandlerAdapter 执行完 handler 后, 调用 postHandle
	解析视图并渲染到response之后, 调用 afterCompletion
	如果中途出现异常, 或 preHandle 提前结束, 则也调用 afterCompletion

总结
	DispatcherServlet 去调用 HandlerExecutionChian 去调用 拦截器具体方法. 
	复杂点是添加一个拦截器到被加入到 HandlerExecutionChian 比较复杂一点, 以及带路径匹配的拦截器实现略复杂一些.
```



### 默认的 HandlerMapping/HandlerAdapter 在何时加入?

```java
org.springframework.web.servlet.DispatcherServlet#initStrategies
protected void initStrategies(ApplicationContext context) {
  // 加载文件上传处理 bean
  // 加载国际化处理 bean
  // 加载主题切换处理 bean
  // 加载 HandlerMapping bean
  // 加载 HandlerAdapter bean
  // 加载异常处理 bean
  // 加载 HttpServletRequest 转视图名称处理策略 bean
  // 加载视图解析器
  // 加载 FlashMap 管理 bean

  // 每一个 init 的逻辑都是类似的, 先从容器中根据 Xxx.class getBean
  // 若无, 则调用 getDefaultStrategy() 从 DispatcherServlet.properties 配置文件中读取
  initMultipartResolver(context);
  initLocaleResolver(context);
  initThemeResolver(context);
  initHandlerMappings(context);
  initHandlerAdapters(context);
  initHandlerExceptionResolvers(context);
  initRequestToViewNameTranslator(context);
  initViewResolvers(context);
  initFlashMapManager(context);
}
```



### 参数是如何与 HTTP 请求 body 绑定的(序列化, 格式化, 绑定)?

```java
1.在适配器执行 handler 的时候, 即 RequestMappingHandlerAdapter#invokeHandlerMethod
2.此方法会进行一些其他处理, 然后准备执行方法前解析参数, 即在 InvocableHandlerMethod#getMethodArgumentValues() 中
3.这个方法中会遍历每一个参数, 再遍历配置的所有 resolvers, 通过 supportsParameter 接口判断是否支持参数解析, 是则调用 resolveArgument 接口获得实参
4.这其中, 最重要的是 resolvers, 其一般在 RequestMappingHandlerAdapter#getDefaultArgumentResolvers() 中添加默认和用户自定义的 resoloves.
5.如 RequestResponseBodyMethodProcessor#resolveArgument() 用于处理带 @RequestBody 注解的参数.  
```

> 总结: 适配器执行具体方法前, 先用反射获取这个方法的 参数(形参)集合, 挨个遍历从 resolvers 找支持解析的类来解析, 得到的返回值作为实参先存起来, 最后调用具体方法时就可以带上实参们执行就实现了将 HTTP 的数据绑定到 controller 的方法参数上的功能.
>
> 参考 RequestResponseBodyMethodProcessor#resolveArgument()

```java
@Override
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                              NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
  // 调用 readWithMessageConverters 读取 body 的数据, 再序列化 json 成相应的 Java Bean
  // 使用 binder 检查 arg 的值是否与 @Valid 的那些注解相关的规则相符, 若有错误, 则抛异常.

  parameter = parameter.nestedIfOptional();
  Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
  String name = Conventions.getVariableNameForParameter(parameter);

  if (binderFactory != null) {
    WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
    if (arg != null) {
      validateIfApplicable(binder, parameter);
      if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
        throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
      }
    }
    if (mavContainer != null) {
      mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
    }
  }

  return adaptArgumentIfNecessary(arg, parameter);
}
```



### 参数校验是如何进行的?

```java
// 参考上面的代码
if (binderFactory != null) {
  WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
  if (arg != null) {
    validateIfApplicable(binder, parameter);
    if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
      throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
    }
  }
  if (mavContainer != null) {
    mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
  }
}
// 这段代码 validateIfApplicable() 就进行了参数校验, 代码如下:
// 逻辑是: 遍历参数所有的注解, 包含 validatedAnn 或以 Valid 开头的注解都进行校验
protected void validateIfApplicable(WebDataBinder binder, MethodParameter parameter) {
  Annotation[] annotations = parameter.getParameterAnnotations();
  for (Annotation ann : annotations) {
    Validated validatedAnn = AnnotationUtils.getAnnotation(ann, Validated.class);
    if (validatedAnn != null || ann.annotationType().getSimpleName().startsWith("Valid")) {
      Object hints = (validatedAnn != null ? validatedAnn.value() : AnnotationUtils.getValue(ann));
      Object[] validationHints = (hints instanceof Object[] ? (Object[]) hints : new Object[] {hints});
      binder.validate(validationHints);
      break;
    }
  }
}

// 然后是 validate 具体的对象.
// 逻辑是: 遍历所有的 validators, 挨个调用 validate 校验
public void validate(Object... validationHints) {
  Object target = getTarget();
  Assert.state(target != null, "No target to validate");
  BindingResult bindingResult = getBindingResult();
  // Call each validator with the same binding result
  for (Validator validator : getValidators()) {
    if (!ObjectUtils.isEmpty(validationHints) && validator instanceof SmartValidator) {
      ((SmartValidator) validator).validate(target, bindingResult, validationHints);
    }
    else if (validator != null) {
      validator.validate(target, bindingResult);
    }
  }
}
// 再往下就没啦... Spring 没有具体的实现, 所以要导入 Hibernate 的啥啥啥包, 这是有原因的.
```





### 有关参数与返回值的一些拦截与干预(@RequestBody, @ResponseBody).

```java
// 参考 RequestMappingHandlerAdapter#afterPropertiesSet()
public void afterPropertiesSet() {
  // Do this first, it may add ResponseBody advice beans
  initControllerAdviceCache();

  if (this.argumentResolvers == null) {
    List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
    this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
  }
  if (this.initBinderArgumentResolvers == null) {
    List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();
    this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
  }
  if (this.returnValueHandlers == null) {
    List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
    this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
  }
}
```

> 由上可知, argumentResolvers/returnValueHandlers 都是此时初始化的, 再点进去看代码发现, 除了默认的, 还有用户自定义的 customArgumentResolvers/customReturnValueHandlers;

```java
// 寻找字段的引用发现
WebMvcConfigurationSupport#addReturnValueHandlers
WebMvcConfigurationSupport#addArgumentResolvers
// 这两个方法可以继承后加入加入自己的 ArgumentResolvers/ReturnValueHandlers, 也就是项目里面继承 WebMvcConfiguration 的那个类, 重写这两个方法, 加入自己的类即可实现对参数/返回值的干预; 常用于统一加密/解密, 记录日志等.
```

> 除此之外, 两边的默认值都有 RequestResponseBodyMethodProcessor, 这就是用于处理@RequestBody/@ResponseBody 的了, 往里面深入的看, 能看到其使用 messageConverters 读取 body 数据, 然后也会在对应的地方触发 advice 的方法.

```java
// 参考 AbstractMessageConverterMethodArgumentResolver#readWithMessageConverters
for (HttpMessageConverter<?> converter : this.messageConverters) {
  Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
  GenericHttpMessageConverter<?> genericConverter =
    (converter instanceof GenericHttpMessageConverter ? (GenericHttpMessageConverter<?>) converter : null);
  if (genericConverter != null ? genericConverter.canRead(targetType, contextClass, contentType) :
      (targetClass != null && converter.canRead(targetClass, contentType))) {
    if (message.hasBody()) {
      HttpInputMessage msgToUse =
        getAdvice().beforeBodyRead(message, parameter, targetType, converterType);
      body = (genericConverter != null ? genericConverter.read(targetType, contextClass, msgToUse) :
              ((HttpMessageConverter<T>) converter).read(targetClass, msgToUse));
      body = getAdvice().afterBodyRead(body, msgToUse, parameter, targetType, converterType);
    }
    else {
      body = getAdvice().handleEmptyBody(null, message, parameter, targetType, converterType);
    }
    break;
  }
}
```







## 参数的N种绑定方式

```java
// 参考 RequestMappingHandlerAdapter#getDefaultArgumentResolvers
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
  List<HandlerMethodArgumentResolver> resolvers = new ArrayList<>();

  // Annotation-based argument resolution
  resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
  resolvers.add(new RequestParamMapMethodArgumentResolver());
  resolvers.add(new PathVariableMethodArgumentResolver());
  resolvers.add(new PathVariableMapMethodArgumentResolver());
  resolvers.add(new MatrixVariableMethodArgumentResolver());
  resolvers.add(new MatrixVariableMapMethodArgumentResolver());
  resolvers.add(new ServletModelAttributeMethodProcessor(false));
  resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
  resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
  resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
  resolvers.add(new RequestHeaderMapMethodArgumentResolver());
  resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
  resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
  resolvers.add(new SessionAttributeMethodArgumentResolver());
  resolvers.add(new RequestAttributeMethodArgumentResolver());

  // Type-based argument resolution
  resolvers.add(new ServletRequestMethodArgumentResolver());
  resolvers.add(new ServletResponseMethodArgumentResolver());
  resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
  resolvers.add(new RedirectAttributesMethodArgumentResolver());
  resolvers.add(new ModelMethodProcessor());
  resolvers.add(new MapMethodProcessor());
  resolvers.add(new ErrorsMethodArgumentResolver());
  resolvers.add(new SessionStatusMethodArgumentResolver());
  resolvers.add(new UriComponentsBuilderMethodArgumentResolver());

  // Custom arguments
  if (getCustomArgumentResolvers() != null) {
    resolvers.addAll(getCustomArgumentResolvers());
  }

  // Catch-all
  resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
  resolvers.add(new ServletModelAttributeMethodProcessor(true));

  return resolvers;
}
```

> 看几个常见的

```bash
1.RequestParamMethodArgumentResolver: 负责解析带 @RequestParam 注解的普通参数
2.RequestParamMapMethodArgumentResolver: 负责解析带 @RequestParam 的 Map 参数....
3.PathVariableMethodArgumentResolver/PathVariableMapMethodArgumentResolver: 同上解析带 @PathVariable 注解的参数
4.RequestResponseBodyMethodProcessor: 负责解析带 @RequestBody 的参数
5.RequestHeaderMethodArgumentResolver: 负责解析带 @RequestHeader 的参数 (表示没用过)
6.ServletRequestMethodArgumentResolver: 负责解析 HttpServletRequest 等类型的参数(即 req, 用的贼多)
7.ServletResponseMethodArgumentResolver: 负责解析 HttpServletResponse 等类型的参数(即 resp, 下载接口没少用)
8.RequestParamMethodArgumentResolver: 啥都解析.... 即不带任何注解的普通类型.
```

