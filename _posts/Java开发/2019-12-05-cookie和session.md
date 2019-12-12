---
layout:     post
title:      cookie和session
subtitle:   cookie和session
date:       2019-12-05
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - 计算机网络

---

# 会话机制

会话机制是Web程序中常用的技术，用来跟踪用户的整个会话，常用的技术是cookie和session。cookie通过在客户端记录信息来确定用户的身份，session通过在服务端记录信息来确定用户的身份。

浏览器给服务器发送请求，访问web程序，该次会话就已经接通，其中不管浏览器发送多少请求(就相当于接通电话后说话一样)，都视为一次会话，直到浏览器关闭，本次会话结束。

# cookie

> cookie由HTTP协议制定，是服务器创建并保存到客户端浏览器的键值对。

#### cookie的工作流程

1. servlet创建cookie，保存少量数据发送给浏览器
2. 浏览器获得cookie数据，保存下来
3. 下次访问时，浏览器发送的请求会携带cookie

#### 特点

1. 一个cookie文件大小4kb，超过4kb浏览器不支持
2. cookie不安全，可能泄露信息
3. 浏览器关闭时cookie销毁
4. 一个服务器最多向一个浏览器保存20个cookie
5. 一个浏览器最多保存300个cookie

#### JavaWeb中使用cookie

**常用方式：**
- 使用response.addCookie()方法向浏览器保存Cookie
- 使用request.getCookies()方法获取浏览器归还的Cookie

原始的方式：
- 使用response发送Set-Cookie响应头
- 使用request获取Cookie请求头

#### cookie的详细信息

在cookie中不只有name和value两个属性。

- maxAge：cookie的最大生命，单位为秒，表示cookie可保存的最大时长。cookie.setMaxAge(60);
    + maxAge > 0 : 浏览器把cookie保存到客户机硬盘，有效时长由maxAge决定
    + maxAge = 0 : cookie只在内存中，浏览器关闭时清除
    + maxAge < 0 : 浏览器会马上删除这个cookie
- cookie的path
    + cookie的path在服务器创建cookie时设置
    + cookie的path并不是这个cookie在客户端的保存路径，而是决定浏览器访问服务器某个路径时要归还哪一个cookie
    + cookie的path的默认值为当前访问路径的父路径
- cookie的domain
    + domain用来指定cookie的域名，当多个二级域中共享cookie时才使用。例如：www.baidu.com与zhidao.baidu.com。
    + 设置domain：cookie.setDomain(".baidu.com");
    + 设置path：cookie.setPath("/");

# session

浏览器请求服务器访问web站点时，服务器在创建session时首先检查这个客户端是否已经有sessionid。如果有了，就直接检索出来用，如果没有就创建一个。sessionid在第一次响应中发回给客户端保存，保存的方式就是cookie。

关闭浏览器不会导致session删除，因此服务器上一般设置有一个session失效时间。

session最大不活动时间默认为30分钟。

#### HttpSession

Servlet三大对象之一，所以有setAttribute(), getAttribute(), removeAttribute()方法。

**HttpSession作用**

- 会话范围：从用户首次访问服务器开始到其关闭浏览器结束
    + 会话：一个用户对服务器的多次连贯请求(中间没有关闭浏览器)
- 服务器会为每个客户端创建一个session对象保存在一个Map中，好比客户在服务器上的账户。这个Map称为session缓存。
    + Servlet中得到session对象：HttpSession session = request.getSession();

#### HttpSession原理

request,getSession()方法：
- 获取cookie中的sessionId
    + 如果session不存在，创建session并保存，将sessionId保存到cookie中
    + 如果session存在，通过sessionId查找。
        * 如果没找到，创建session并保存，净新创建的sessionId保存到cookie中
        * 如果找到了，就不用新创建了
    + 返回session
- 如果创建了新的session，浏览器会得到一个包含sessionId的cookie。其生命为-1，只保存在内存中。
- 下次请求时因为可以通过cookie中的sessionId找到session对象，所以与上次请求使用的是同一个session

通过request.getSession()方法第一次获取session时，服务器创建session。

web.xml中配置session的最大不活动时间
```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

#### URL重写

session依赖于cookie，因为客户端发出请求时归还cookie中包含sessionId才能找到对应的session。如果客户端禁用了cookie，session也就没用了。

response.encodeURL(String url)方法会对url进行智能重写，当请求中没有归还sessionId这个cookie，该方法会重写url来替代cookie(在该网站的所有超链接和表单中都添加一个特殊的请求参数sessionId，这样服务器就能够找到session对象)。

# session和cookie的区别

1. cookie存放在客户端，session存放在服务器
2. cookie的安全性较低，考虑安全性要使用session
3. 由于session存放在服务器上，要减轻服务器压力应该使用cookie
4. 单个cookie大小限制为4k，一个站点最多保存20个cookie