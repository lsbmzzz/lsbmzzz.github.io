---
layout:     post
title:      SpringBoot web笔记
subtitle:   SpringBoot web笔记
date:       2021-02-03
author:     张三
header-img: img/spring.jpg
catalog: true
mathjax: true
tags:
    - SpringBoot

---

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

## 欢迎页

将名为index.html的文件放在静态资源路径下，访问根路径可自动跳转。

可以配置静态资源路径，但 **不能配置静态资源访问前缀**。

## 标签图标

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

### 欢迎页的配置同理，在WebMvcAutoConfigurationAdapter下有一个welcomePageHandlerMapping方法
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

## 请求映射

在Controller上使用@RequestMapping注解

Rest风格原
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

    @RequestMapping("/bug.jpg")
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
