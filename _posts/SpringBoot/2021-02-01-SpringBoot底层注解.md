---
layout:     post
title:      SpringBoot底层注解
subtitle:   SpringBoot底层注解
date:       2021-02-01
author:     张三
header-img: img/spring.jpg
catalog: true
mathjax: true
tags:
    - SpringBoot

---

# @Configuration

作用：告诉springboot这是一个配置类，配置类中可使用@Bean标注在方法上给容器注册组件

```java
@Configuration(proxyBeanMethods = false) 
public class MyConfig {

    /**
     * 对象是单例
     * @Bean给容器添加组件，方法名默认为组件id，返回类型为组件类型，值为组件在容器中的实例
     */
    @Bean 
    public User user01(){
        User zhangsan = new User("zhangsan", 18);
        //user组件依赖了Pet组件
        zhangsan.setPet(tomcatPet());
        return zhangsan;
    }
    // 自定义组件id
    @Bean("tom")
    public Pet tomcatPet(){
        return new Pet("tomcat");
    }
}
```

- 注册的组件默认是单例
- proxyBeanMethods：是不是代理Bean的方法
	+ 默认是true，获得代理对象，总会检查容器中是否已有该组件
    + false时，不再检查该组件是否已存在


# @Import

给容器中自动创建组件，默认名字是全类名

```java
@Import({User.class, Resident.class})
@Configuration(proxyBeanMethods = false)
public class Cell{

}
```


# @Conditional

满足Conditional指定的条件时，才进行组件注入

![Conditional注解](/img/SpringBoot/Conditional注解.png)


# @ImportResource

导入资源

导入beans.xml案例：
```java
@ImportResource("classpath:beans.xml")
```

好处：不需要重写老的资源文件


# @ConfigurationProperties

- 使用Java读取properties文件内容，并封装到JavaBean中
- 只有容器中的组件才可以使用

```java
@Component
@ConfigurationProperties(prefix = "demo")
public class Demo {

}
```

```java
@EnableConfigurationProperties(Demo.class) // 开启Demo的属性配置功能，并将该组件自动注册到容器中
public class MyConfig {

}
```

# @SpringBootApplication 自动配置

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{

}
```

### @SpringBootConfiguration

代表当前是一个配置类

### @ComponentScan

指定要扫描的范围

### @SpringBootConfiguration

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {

}
```

##### @AutoConfigurationPackage

AutoConfigurationPackage：利用Register给容器批量导入组件
将Main程序所在包下的所有组件导入进来
```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    
}

```

Register:
```java
    static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        Registrar() {
        }

        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
        }

        public Set<Object> determineImports(AnnotationMetadata metadata) {
            return Collections.singleton(new AutoConfigurationPackages.PackageImports(metadata));
        }
    }
```

##### @Import({AutoConfigurationImportSelector.class})

1. 利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
2. 调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
3. 利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4. 从META-INF/spring.factories位置来加载一个文件。
    + 默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
    + spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories

127个场景的所有自动配置启动的时候默认全部加载,但按照@Conditional，最终会按需配置。

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定
- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了
- 定制化配置
    + 用户直接自己@Bean替换底层的组件
    + 用户去看这个组件是获取的配置文件什么值就去修改。

**xxxxxAutoConfiguration -> 组件 -> xxxxProperties里面拿值 -> application.properties**
