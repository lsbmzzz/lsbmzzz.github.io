---
layout:     post
title:      Thread，Runnable与Callable
subtitle:   Thread，Runnable与Callable
date:       2019-11-24
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - 多线程
    - Java

---

# Thread与Runnable的区别

Thread类定义在java.lang包中，一个类只要继承Thread并覆写run()即可实现多线程操作。但在java中只支持单继承。相比之下，Runnable接口更加灵活。

Thread类的定义
```java
public class Thread extends Object implements Runnable{}
```

Thread类也是Runnable接口的子类，所以继承Thread类覆写的也是Runnable接口。

多线程设计中使用了代理设计模式结构，用户定义的线程只负责核心功能，其他的全部交给Thread类。

Thread启动多线程时调用的是start()，然后找到的是run()。通过Thread类传递了一个Runnable接口的对象时，该接口的对象会被Thread中的target属性保存。在start()执行时，会调用Thread中的run()，而这个run()会去调用Runnable接口子类被覆写过的run()。

多线程开发的本质在于多个线程可以进行同一个资源的抢占，Thread描述线程，Runnable描述资源。

Thread会产生若干个线程对象(线程)，而Runnable的子类描述资源，所有线程对象要访问Runnable描述的资源。

# Callable

Runnable的缺点是无法获得返回值，所以从JDK1.5后在java.util.concurrent增加了新的线程实现接口Callable。

```java
@FunctionalInterface
public interface Callable<V>{
    public V call() throws Exception;
}
```

泛型的类型就是返回的数据类型。

![Callable](/img/Java/Callable.png)

FutureTask<V>继承RunnableFuture<V>

RunnableFuture继承了Runnable和Future<V>

Future<V>中有一个get()方法(Runnable无法获得返回值，通过Future中的get()来实现返回值接收。)

Callable封装在FutureTask中传递给Thread类

线程实现类覆写call()即可。