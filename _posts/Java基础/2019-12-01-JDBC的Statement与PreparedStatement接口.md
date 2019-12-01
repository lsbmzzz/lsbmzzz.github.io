---
layout:     post
title:      JDBC的Statement与PreparedStatement接口
subtitle:   JDBC的Statement与PreparedStatement接口
date:       2019-12-01
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - Java

---

当获取了java.sql.Connection接口对象后，那么其核心的目的一定是为了进行数据库的操作，数据库的开发操作应该使用标准sql语句来完成，所以需要有一个SQL的执行器。执行器就可以利用Statement接口完成。

# Statement

定义：
```java
public interface Statement extends Wrapper, AutoCloseable
```

该接口是AutoCloseable子接口，所以可以得出结论：每一次进行数据库操作完成后，都应该关闭Statement操作，即一条SQL的执行一定是一个Statement接口对象。想要获取Statement接口对象，那么必须依靠Connection接口提供的方法完成。

- 获取Statement对象：Statement createStatement()

![Statemet接口](/img/Java基础/Statemet接口.png)

获取了Statement接口对象之后，就可以使用SQL进行处理了。Statement中提供了两个方法的支持：
|方法|功能|
|---|---|
|int executeUpdate(String sql)|数据库更新(insert, update, delete)|
|ResultSet executeQuery(String sql)|数据库查询(select)|

两个数据库的操作方法中都需要接收SQL字符串，也就是说Statement接口可以直接使用SQL语句实现开发。

数据更新处理返回的是SQL执行后的影响行数。

ResultSet对象时保存在内存中的，如果说查询数据的返回结果过大，那么程序可能会出现性能上或其他方面的问题。

#### Statement接口操作的问题

利用Statement执行的SQL语句问题有如下三种：
- 无法很好地描述日期类型
- 需要进行SQL语句拼接，导致SQL语句编写和维护困难
- 对敏感字符无法进行合理拼凑
- SQL注入漏洞

# PreparedStatement

PreparedStatement是Statement的一个子接口，用于解决Statement存在的问题。这个接口最大的好处是可以编写正常的SQL（数据不再和SQL语法混合在一起），同时利用占位符形式，在SQL正常执行完毕后可以进行数据的设置。

定义：
```java
public interface PreparedStatement extends Statement
```

![PreparedStatemet接口](/img/Java基础/PreparedStatemet接口.png)

获得PreparedStatement接口的实例，依然需要通过Connection接口来实现创建方法：
- PreparedStatement prepareStatement(String sql)

由于SQL语句已经在创建PreparedStatement接口对象时提供了，所以在执行数据库操作时也要更换方法：

|方法|功能|
|---|---|
|int executeUpdate()|数据库更新(insert, update, delete)|
|ResultSet executeQuery()|数据库查询(select)|

在JDBC中不管使用的是PreparedStatement设置的日期时间还是使用ResultSet获取的日期时间实际上都是java.util.Date的子类，也就是说是如下的对应关系。

![PreparedStatemet与Date](/img/Java基础/PreparedStatemet与Date.png)