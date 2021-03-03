---
layout:     post
title:      SpringBoot2 web笔记
subtitle:   SpringBoot2 web笔记
date:       2021-02-03
author:     张三
header-img: img/spring.jpg
catalog: true
mathjax: true
tags:
    - SpringBoot

---

> [雷丰阳2021版SpringBoot2零基础入门springboot全套完整版（spring boot2）](https://www.bilibili.com/video/BV19K4y1L7MT)

# WEB 场景

## 静态资源

### 静态资源路径

把静态资源放在

- /static 
- /public
- /resources
- /META-INF/resources

下，使用根目录+资源名就可以直接访问。

### 静态资源访问配置

可通过yml配置增加访问静态资源的前缀或改变静态资源路径：
```yaml
spring: 
    # 增加访问静态资源的前缀
    mvc: 
        static-path-pattern: /resources/**
    # 改变静态资源路径
    resources: 
        static-locations: classpath:/newpath
```

**原理：请求到达后，首先检查Controller能不能处理，如果不能，就交给静态资源处理器。如果静态资源也无法处理，返回404页面。**

### 欢迎页

将名为index.html的文件放在静态资源路径下，访问根路径可自动跳转。

可以配置静态资源路径，但 **不能配置静态资源访问前缀**。

### 标签图标

将favicon.ico放在静态资源目录下。

## 静态资源配置原理

- 默认加载 org\springframework\boot\spring-boot-autoconfigure\2.4.0\spring-boot-autoconfigure-2.4.0.jar!\org\springframework\boot\autoconfigure\web\servlet 下的 xxxAutoConfiguration 类
- WebMvcAutoConfiguration 类 生效
```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
        ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

}
```
- WebMvcAutoConfiguration 中有一个内部类 WebMvcAutoConfigurationAdapter
    + WebMvcProperties 和 spring.mvc 进行了绑定
    + ResourceProperties 和 spring.resources 进行了绑定
```java
    @Configuration(proxyBeanMethods = false)
    @Import(EnableWebMvcConfiguration.class)
    @EnableConfigurationProperties({ WebMvcProperties.class,
            org.springframework.boot.autoconfigure.web.ResourceProperties.class, WebProperties.class })
    @Order(0)
    public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {

    }
```
- WebMvcAutoConfigurationAdapter中只有一个带参构造器，所有参数的值都从容器中确定
```java
        public WebMvcAutoConfigurationAdapter(
                org.springframework.boot.autoconfigure.web.ResourceProperties resourceProperties,   // 获取和spring.resources绑定的所有的值的对象
                WebProperties webProperties,    
                WebMvcProperties mvcProperties,     // 获取和spring.mvc绑定的所有的值的对象
                ListableBeanFactory beanFactory,    // beanFactory Spring的beanFactory
                ObjectProvider<HttpMessageConverters> messageConvertersProvider,    //找到所有的HttpMessageConverters
                ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,    // 找到 资源处理器的自定义器
                ObjectProvider<DispatcherServletPath> dispatcherServletPath,    // DispatcherServlet处理的路径
                ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {  //给应用注册Servlet、Filter等
            this.resourceProperties = resourceProperties.hasBeenCustomized() ? resourceProperties
                    : webProperties.getResources();
            this.mvcProperties = mvcProperties;
            this.beanFactory = beanFactory;
            this.messageConvertersProvider = messageConvertersProvider;
            this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
            this.dispatcherServletPath = dispatcherServletPath;
            this.servletRegistrations = servletRegistrations;
            this.mvcProperties.checkConfiguration();
        }
```
- 资源处理的默认规则
    + 当this.resourceProperties.isAddMappings()为false时，所有静态资源配置被禁用
```java
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
                return;
            }
            Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
            CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
            if (!registry.hasMappingForPattern("/webjars/**")) {
                customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                        .addResourceLocations("classpath:/META-INF/resources/webjars/")
                        .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl)
                        .setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));
            }
            String staticPathPattern = this.mvcProperties.getStaticPathPattern();
            if (!registry.hasMappingForPattern(staticPathPattern)) {
                customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                        .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                        .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl)
                        .setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));
            }
        }
```
- 所以，禁用静态资源：
```yaml
spring:
  resources:
    add-mappings: false   # 禁用所有静态资源规则
```
- 静态资源的4个目录：
    + WebProperties 下有 Resources 方法，CLASSPATH_RESOURCE_LOCATIONS 指定了四个位置
```java

    public static class Resources {

        private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
                "classpath:/resources/", "classpath:/static/", "classpath:/public/" };
        ...
    }
```

**欢迎页的配置同理，在WebMvcAutoConfigurationAdapter下有一个welcomePageHandlerMapping方法**

```java
        @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
                FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
            WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
                    new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
                    this.mvcProperties.getStaticPathPattern());
            welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
            welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
            return welcomePageHandlerMapping;
        }
```

其中 new WelcomePageHandlerMapping:

```java
WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
            ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
        if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {   // 这里写死了/**，所以静态资源加前缀后无法访问到
            logger.info("Adding welcome page: " + welcomePage.get());
            setRootViewName("forward:index.html");
        }
        else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            logger.info("Adding welcome page template: index");
            setRootViewName("index");
        }
    }
```

# 请求处理

在Controller上使用@RequestMapping注解

## Rest风格的请求映射

关键Filter：HiddenHttpMethodFilter

- 表单method=post，隐藏域 _method=put
- 需要在SpringBoot中手动开启
```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
```

- 表单提交会带上_method=PUT
- 请求过来被HiddenHttpMethodFilter拦截
- 请求是否正常，并且是POST
    + 获取到_method的值。
    + 兼容以下请求；PUT.DELETE.PATCH
    + 原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值。
    + 过滤器链放行的时候用wrapper。以后的方法调用getMethod是调用requesWrapper的。
可以直接使用GetMapping、PostMapping、PutMapping、DeleteMapping

```java
@RestController
public class HelloController {

    @RequestMapping("/zhangsan.jpg")
    public String hello(){
        //request
        return "aaaa";
    }

//    @RequestMapping(value = "/user",method = RequestMethod.GET)
    @GetMapping("/user")
    public String getUser(){
        return "GET-张三";
    }

//    @RequestMapping(value = "/user",method = RequestMethod.POST)
    @PostMapping("/user")
    public String saveUser(){
        return "POST-张三";
    }

//    @RequestMapping(value = "/user",method = RequestMethod.PUT)
    @PutMapping("/user")
    public String putUser(){
        return "PUT-张三";
    }

    @DeleteMapping("/user")
//    @RequestMapping(value = "/user",method = RequestMethod.DELETE)
    public String deleteUser(){
        return "DELETE-张三";
    }
```

## 请求映射的原理

![](/img/后端开发/SpringBoot/Spring请求调用原理.png)

- HttpServlet 中的 doGet 在 FrameWorkServlet 中 调用本类的 processRequest 方法
- processRequest 调用本类的 doService 方法
- doService 在子类 DispatcherServlet 中进行了实现
- doService 调用 DispatcherServlet 的 doDispatch 方法

doDispatch 方法：

```java
    /**
     * Process the actual dispatching to the handler.
     * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
     * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
     * to find the first that supports the handler class.
     * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
     * themselves to decide which methods are acceptable.
     * @param request current HTTP request
     * @param response current HTTP response
     * @throws Exception in case of any kind of processing failure
     */
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = (processedRequest != request);

                // 决定哪一个handler可以处理当前请求
                mappedHandler = getHandler(processedRequest);
                if (mappedHandler == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }

                // Determine handler adapter for the current request.
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

                // Actually invoke the handler.
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }

                applyDefaultViewName(processedRequest, mv);
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
            catch (Throwable err) {
                // As of 4.3, we're processing Errors thrown from handler methods as well,
                // making them available for @ExceptionHandler methods and other scenarios.
                dispatchException = new NestedServletException("Handler dispatch failed", err);
            }
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Throwable err) {
            triggerAfterCompletion(processedRequest, response, mappedHandler,
                    new NestedServletException("Handler processing failed", err));
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            }
            else {
                // Clean up any resources used by a multipart request.
                if (multipartRequestParsed) {
                    cleanupMultipart(processedRequest);
                }
            }
        }
    }
```

handlerMapping 中有

- RequestMappingHandlerMapping : 保存了所有的 handler 映射
- WelcomePageHandlerMapping : 配置了欢迎页 index.html
- BeanNameUrlHandlerMapping
- RouterFunctionMapping
- SimpleUrlHandlerMapping

```java
/** List of HandlerMappings used by this servlet. */
    @Nullable
    private List<HandlerMapping> handlerMappings;
```

请求到达后，会遍历所有HandlerMapping查找是否有其请求信息

## 请求处理方式

### 注解

- @PathVariable（路径变量）
- @RequestHeader（获取请求头）
- @RequestParam（获取请求参数）
- @CookieValue（获取cookie值）

```java
    //  car/2/owner/zhangsan
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     @PathVariable Map<String,String> pv, // 所有路径变量
                                     @RequestHeader("User-Agent") String userAgent,
                                     @RequestHeader Map<String,String> header, // 所有请求头
                                     @RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     @RequestParam Map<String,String> params, // 所有请求参数
                                     @CookieValue("_ga") String _ga,
                                     @CookieValue("_ga") Cookie cookie){ // 完整cookie
        // map.pub()...
        Map<String,Object> map = new HashMap<>();
        return map;
```

- @RequestBody（获取请求体[POST]）

```java
    @PostMapping("/save")
    public Map postMethod(@RequestBody String content){
        Map<String,Object> map = new HashMap<>();
        map.put("content",content);
        return map;
    }
```

- @RequestAttribute (获取request请求域属性中的值)

```java
    request.setAttribute("msg", "message");
    request.getAttribute("msg");
```

- @MatrixVariable (矩阵变量)
    + 多个变量之间使用;分隔
```java
    /cars/sell;low=34;brand=byd,audi,yd
```
    + 一个变量的多个值使用,分隔，或是使用重复变量名
```java
    color=red,green,blue

    color=red;color=green;color=blue
```
    + 当不同路径下的变量名相同时，使用pathVar，指定获取哪个路径下的值
    + SpringBoot默认禁用矩阵变量，需要手动开启
        * 手动开启：原理。对于路径的处理。UrlPathHelper进行解析。removeSemicolonContent（移除分号内容）支持矩阵变量的
    + 矩阵变量必须有url路径变量才能被解析

#### 注解方式的原理

- DispatcherServlet开始，进入doDispatch方法
- 在 HandlerMapping 中找到能处理请求的Handler，并为其找到一个适配器 HandlerAdapter
- RequestMappingHandlerAdapter中的handleInternal调用invokeHandlerMethod执行目标方法并确定方法参数的值

![HandlerAdapter.png](/img/后端开发/SpringBoot/HandlerAdapter.png)

- 4种 Adapter 前两种用的最多
    + 支持方法上标注@RequestMapping注解
    + 支持函数式编程

**doDispatch方法：** 找到能处理请求的Handler，并为其找到一个适配器 HandlerAdapter

```java
    /**
     * Process the actual dispatching to the handler.
     * <p>The handler will be obtained by applying the servlet's HandlerMappings in order.
     * The HandlerAdapter will be obtained by querying the servlet's installed HandlerAdapters
     * to find the first that supports the handler class.
     * <p>All HTTP methods are handled by this method. It's up to HandlerAdapters or handlers
     * themselves to decide which methods are acceptable.
     * @param request current HTTP request
     * @param response current HTTP response
     * @throws Exception in case of any kind of processing failure
     */
    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = (processedRequest != request);

                // Determine handler for the current request.
                mappedHandler = getHandler(processedRequest);
                if (mappedHandler == null) {
                    noHandlerFound(processedRequest, response);
                    return;
                }

                // Determine handler adapter for the current request.
                // 重点
                HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

                // Process last-modified header, if supported by the handler.
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }

                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }

                // Actually invoke the handler.
                // 重点 执行目标方法
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }

                applyDefaultViewName(processedRequest, mv);
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            }
            catch (Exception ex) {
                dispatchException = ex;
            }
            catch (Throwable err) {
                // As of 4.3, we're processing Errors thrown from handler methods as well,
                // making them available for @ExceptionHandler methods and other scenarios.
                dispatchException = new NestedServletException("Handler dispatch failed", err);
            }
            processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
        }
        catch (Exception ex) {
            triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
        }
        catch (Throwable err) {
            triggerAfterCompletion(processedRequest, response, mappedHandler,
                    new NestedServletException("Handler processing failed", err));
        }
        finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                // Instead of postHandle and afterCompletion
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            }
            else {
                // Clean up any resources used by a multipart request.
                if (multipartRequestParsed) {
                    cleanupMultipart(processedRequest);
                }
            }
        }
    }
```

**handleInternal：** 执行目标方法

```java
    protected ModelAndView handleInternal(HttpServletRequest request,
            HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

        ModelAndView mav;
        checkRequest(request);

        // Execute invokeHandlerMethod in synchronized block if required.
        if (this.synchronizeOnSession) {
            HttpSession session = request.getSession(false);
            if (session != null) {
                Object mutex = WebUtils.getSessionMutex(session);
                synchronized (mutex) {
                    mav = invokeHandlerMethod(request, response, handlerMethod);
                }
            }
            else {
                // No HttpSession available -> no mutex necessary
                mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            // No synchronization on session demanded at all...
            // 执行目标方法
            mav = invokeHandlerMethod(request, response, handlerMethod);
        }

        if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
            if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
                applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
            }
            else {
                prepareResponse(response);
            }
        }

        return mav;
    }
```

**invokeHandlerMethod：** 执行目标方法

```java
    /**
     * Invoke the {@link RequestMapping} handler method preparing a {@link ModelAndView}
     * if view resolution is required.
     * @since 4.2
     * @see #createInvocableHandlerMethod(HandlerMethod)
     */
    @Nullable
    protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
            HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

        ServletWebRequest webRequest = new ServletWebRequest(request, response);
        try {
            WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
            ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

            ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
            if (this.argumentResolvers != null) {
                invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
            }
            if (this.returnValueHandlers != null) {
                invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
            }
            invocableMethod.setDataBinderFactory(binderFactory);
            invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

            ModelAndViewContainer mavContainer = new ModelAndViewContainer();
            mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
            modelFactory.initModel(webRequest, mavContainer, invocableMethod);
            mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

            AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
            asyncWebRequest.setTimeout(this.asyncRequestTimeout);

            WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
            asyncManager.setTaskExecutor(this.taskExecutor);
            asyncManager.setAsyncWebRequest(asyncWebRequest);
            asyncManager.registerCallableInterceptors(this.callableInterceptors);
            asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

            if (asyncManager.hasConcurrentResult()) {
                Object result = asyncManager.getConcurrentResult();
                mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
                asyncManager.clearConcurrentResult();
                LogFormatUtils.traceDebug(logger, traceOn -> {
                    String formatted = LogFormatUtils.formatValue(result, !traceOn);
                    return "Resume with async result [" + formatted + "]";
                });
                invocableMethod = invocableMethod.wrapConcurrentResult(result);
            }
            // 重点 
            invocableMethod.invokeAndHandle(webRequest, mavContainer);
            if (asyncManager.isConcurrentHandlingStarted()) {
                return null;
            }

            return getModelAndView(mavContainer, modelFactory, webRequest);
        }
        finally {
            webRequest.requestCompleted();
        }
    }
```

**invokeAndHandle**

- 通过 invokeForRequest 执行目标方法

```java
    /**
     * Invoke the method and handle the return value through one of the
     * configured {@link HandlerMethodReturnValueHandler HandlerMethodReturnValueHandlers}.
     * @param webRequest the current request
     * @param mavContainer the ModelAndViewContainer for this request
     * @param providedArgs "given" arguments matched by type (not resolved)
     */
    public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {

        Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs); // 执行目标方法
        setResponseStatus(webRequest);

        if (returnValue == null) {
            if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
                disableContentCachingIfNecessary(webRequest);
                mavContainer.setRequestHandled(true);
                return;
            }
        }
        else if (StringUtils.hasText(getResponseStatusReason())) {
            mavContainer.setRequestHandled(true);
            return;
        }

        mavContainer.setRequestHandled(false);
        Assert.state(this.returnValueHandlers != null, "No return value handlers");
        try {
            this.returnValueHandlers.handleReturnValue(
                    returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
        }
        catch (Exception ex) {
            if (logger.isTraceEnabled()) {
                logger.trace(formatErrorForReturnValue(returnValue), ex);
            }
            throw ex;
        }
    }
```

##### 参数解析器
确定将要执行的目标方法的每一个参数的值是什么;
SpringMVC目标方法能写多少种参数类型。取决于参数解析器。

![26个参数解析器](/img/后端开发/SpringBoot/HandlerMethodArgumentResolver.png)

##### 返回值处理器
可以处理哪些类型的返回值

![15个返回值处理器](/img/后端开发/SpringBoot/HandlerMethodReturnValueHandler.png)

### Servlet API

ServletRequestMethodArgumentResolver 下的 supportsParameter 方法中出现的均支持

```java
    public boolean supportsParameter(MethodParameter parameter) {
        Class<?> paramType = parameter.getParameterType();
        return (WebRequest.class.isAssignableFrom(paramType) ||
                ServletRequest.class.isAssignableFrom(paramType) ||
                MultipartRequest.class.isAssignableFrom(paramType) ||
                HttpSession.class.isAssignableFrom(paramType) ||
                (pushBuilder != null && pushBuilder.isAssignableFrom(paramType)) ||
                (Principal.class.isAssignableFrom(paramType) && !parameter.hasParameterAnnotations()) ||
                InputStream.class.isAssignableFrom(paramType) ||
                Reader.class.isAssignableFrom(paramType) ||
                HttpMethod.class == paramType ||
                Locale.class == paramType ||
                TimeZone.class == paramType ||
                ZoneId.class == paramType);
    }
```

# 响应处理

方法上注解 @ResponseBody 可以自动处理返回json数据

![15个返回值解析器（处理器）](/img/SpringBoot/HandlerMethodReturnValueHandler.png)

源码过程：

- ServletInvocableHandlerMethod 下的 invokeAndHandle 方法：

```java
    /**
     * Invoke the method and handle the return value through one of the
     * configured {@link HandlerMethodReturnValueHandler HandlerMethodReturnValueHandlers}.
     * @param webRequest the current request
     * @param mavContainer the ModelAndViewContainer for this request
     * @param providedArgs "given" arguments matched by type (not resolved)
     */
    public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {

        Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
        setResponseStatus(webRequest);

        if (returnValue == null) {
            if (isRequestNotModified(webRequest) || getResponseStatus() != null || mavContainer.isRequestHandled()) {
                disableContentCachingIfNecessary(webRequest);
                mavContainer.setRequestHandled(true);
                return;
            }
        }
        else if (StringUtils.hasText(getResponseStatusReason())) {
            mavContainer.setRequestHandled(true);
            return;
        }

        mavContainer.setRequestHandled(false);
        Assert.state(this.returnValueHandlers != null, "No return value handlers");
        try {
            // 处理返回值
            this.returnValueHandlers.handleReturnValue(
                    returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
        }
        catch (Exception ex) {
            if (logger.isTraceEnabled()) {
                logger.trace(formatErrorForReturnValue(returnValue), ex);
            }
            throw ex;
        }
    }
```

- 进入 HandlerMethodReturnValueHandlerComposite 的 handleReturnValue 处理返回值

```java
    /**
     * Iterate over registered {@link HandlerMethodReturnValueHandler HandlerMethodReturnValueHandlers} and invoke the one that supports it.
     * @throws IllegalStateException if no suitable {@link HandlerMethodReturnValueHandler} is found.
     */
    @Override
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
            ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
        // 拿到返回值对象和返回值类型，找哪个handler可以处理它
        HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
        if (handler == null) {
            throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
        }
        // 进行处理
        handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
    }
```

- 查找可以处理返回值的handler的方法：

```java
    @Nullable
    private HandlerMethodReturnValueHandler selectHandler(@Nullable Object value, MethodParameter returnType) {
        boolean isAsyncValue = isAsyncReturnValue(value, returnType);
        for (HandlerMethodReturnValueHandler handler : this.returnValueHandlers) {
            if (isAsyncValue && !(handler instanceof AsyncHandlerMethodReturnValueHandler)) {
                continue;
            }
            if (handler.supportsReturnType(returnType)) {
                return handler;
            }
        }
        return null;
    }
```

- 返回值处理器的解析步骤：

1. 返回值处理器先判断是否支持这种类型 supportsReturnType
2. 如果支持，调用handleReturnValue进行处理
3. RequestResponseBodyMethodProcessor 可以利用 MessageConverters 进行处理返回值标了 @ResponseBody 注解的方法，将数据写为 json。
    1. 内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）
    2. 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据，
    3. SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？
        1. 得到MappingJackson2HttpMessageConverter可以将对象写为json
        2. 利用MappingJackson2HttpMessageConverter将对象转为json再写出去。

## 内容协商

根据客户端接受能力不同，返回不同媒体类型的数据

需要引入依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

开启基于请求参数的内容协商功能 

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: true
```

(请求参数携带format=xml即可指定返回xml数据)

**原理**

1. 检查当前的响应头中是否已经有了确定的媒体类型
2. 根据Accept请求头字段，获取客户端支持的内容类型
3. 遍历MessageConvertre，查找谁可以支持当前对象
4. 找到支持当前对象的converter，得到converter支持度媒体类型
5. 找到最佳匹配的媒体类型，用它进行转化

# 拦截器

HandlerInterceptor接口中定义了三个方法：

- preHandler 在目标方法之前进行调用
- postHandler 在目标方法处理后，到达页面之前调用
- afterHandler 整个请求处理完成后调用

**使用**

1. 拦截器实现HandlerInterceptor接口
2. 通过实现WebMvcConfiguer中的addInterceptions 将拦截器注册到容器中
3. 指定拦截规则（注意放行静态资源）

执行原理：

1. 根据当前请求，找到HandlerExecutionChain【可以处理请求的handler以及handler的所有 拦截器】
2. 先执行 所有拦截器的 preHandle方法
    - 如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
    - 如果当前拦截器返回为false。直接倒序执行所有已经执行了的拦截器的  afterCompletion
3. 如果任何一个拦截器返回false。直接跳出不执行目标方法
4. 所有拦截器都返回True。执行目标方法
5. 倒序执行所有拦截器的postHandle方法。
6. 前面的步骤有任何异常都会直接倒序触发 afterCompletion
7. 页面成功渲染完成以后，也会倒序触发 afterCompletion

# 错误处理

SpringBoot默认提供 /error 来处理所有错误映射
/error/下的4xx，5xx页面会被自动解析
先匹配具体错误码，匹配不到会跳转 4xx，5xx 页面

### 走动配置原理

自动配置类 ErrorMvcAutoConfiguration 

- 组件 DefaultErrorAttributes errorAttributes
    + public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered 定义错误页面中包含哪些数据（exception, trace, message, errors 等）
- 组件 BasicErrorController basicErrorController （响应json或白页）
    + public class BasicErrorController extends AbstractErrorController
        * public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) 处理默认的error请求
    + public View defaultErrorView() 响应默认的错误页面
    + public BeanNameViewResolver beanNameViewResolver() 按照返回的视图名称作为组件，去容器中匹配View对象
- DefaultErrorViewResolver conventionErrorViewResolver 映射错误类型，根据http状态码作为视图地址匹配对应页面

private static class StaticView implements View 定义了默认白页

### 错误处理流程

1. 目标方法发生异常时，被catch，当前请求结束并使用dispatchException处理
2. 进入视图解析流程（页面渲染） private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv, @Nullable Exception exception) throws Exception
3. mv = processHandlerException 处理异常，返回ModelAndView

# 注入原生组件

1. 使用servlet api注入（推荐）

    - 在启动类上 @ServletComponentScan(basePackages = "包名") 指定servlet组件位置
    - 在类上标注：
        + @WebServlet(urlPatterns = "/my") 会直接响应，不经过拦截器
        + @WebFilter(urlPatterns={"/css/*","/images/*"})
        + @WebListener

2. 使用RegistrationBean注入

```java
@Configuration
public class MyResigentrationConfig {
    @Bean
    public ServletRegistrationBean myServlet() {
        MyServlet myServlet = new MyServlet();
        return new ServletRegistrationBean(myServlet, "/my");
    }

    @Bean
    public FilterRegistrationBean myFilter() {
        MyFilter myFilter = new MyFilter();
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/my1", "/my2", "/my3"));
        return filterRegistrationBean;
    }

    @Bean
    public ServletListenerRegistrationBean myListener() {
        MyServletContextListener myListener = new MyServletContextListener();
        ServletListenerRegistrationBean listenerRegistrationBean = new ServletListenerRegistrationBean(myListener);
        return listenerRegistrationBean;
    }
}
```

