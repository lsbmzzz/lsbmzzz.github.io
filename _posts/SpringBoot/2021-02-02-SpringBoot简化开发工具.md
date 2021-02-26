---
layout:     post
title:      SpringBoot简化开发工具Lombok
subtitle:   SpringBoot简化开发工具Lombok
date:       2021-02-02
author:     张三
header-img: img/spring.jpg
catalog: true
mathjax: true
tags:
    - SpringBoot

---

> [雷丰阳2021版SpringBoot2零基础入门springboot全套完整版（spring boot2）](https://www.bilibili.com/video/BV19K4y1L7MT)

# 简化JavaBean

- 在pom.xml文件中导入依赖
```xml
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
```
- 简化JavaBean开发
    + @NoArgsConstructor ： 无参构造器
    + @AllArgsConstructor ： 全参构造器
    + @Data ： Getter 和 Setter方法
    + @ToString ： toString方法
    + @EqualsAndHashCode ： equals和hashCode方法
```java
@NoArgsConstructor
@AllArgsConstructor
@Data
@ToString
@EqualsAndHashCode
public class User {
    private String name;
    private Integer age;
}
```

# 简化日志开发

@Slf4j注解，方法中使用log.info("日志内容");

```java
@Slf4j
@RestController
public class helloController {
    @RequestMapping("/hello")
    public String hello(@RequestParam("name") String name){
        log.info("message");
        return name;
    }
}
```
