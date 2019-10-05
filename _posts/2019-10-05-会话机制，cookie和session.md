---
layout:     post
title:      会话机制，cookie和session
subtitle:   会话机制，cookie和session
date:       2019-10-05
author:     正版慕言
header-img: img/blog_bg_1.jpg
catalog: true
mathjax: true
tags:
    - cookie
    - session

---

# 会话机制

会话机制是Web程序中常用的技术，用来跟踪用户的整个会话，常用的技术是cookie和session。cookie通过在客户端记录信息来确定用户的身份，session通过在服务端记录信息来确定用户的身份。

浏览器给服务器发送请求，访问web程序，该次会话就已经接通，其中不管浏览器发送多少请求(就相当于接通电话后说话一样)，都视为一次会话，直到浏览器关闭，本次会话结束。

# cookie

cookie的工作流程：

1. servlet创建cookie，保存少量数据发送给浏览器
2. 浏览器获得cookie数据，保存下来
3. 下次访问时，浏览器发送的请求会携带cookie

特点：

1. 一个cookie文件大小4kb，超过4kb浏览器不支持
2. cookie不安全，可能泄露信息
3. 浏览器关闭时cookie销毁

# session

浏览器请求服务器访问web站点时，服务器在创建session时首先检查这个客户端是否已经有sessionid。如果有了，就直接检索出来用，如果没有就创建一个。sessionid在第一次响应中发回给客户端保存，保存的方式就是cookie。

关闭浏览器不会导致session删除，因此服务器上一般设置有一个session失效时间。

# session和cookie的区别

1. cookie存放在客户端，session存放在服务器
2. cookie的安全性较低，考虑安全性要使用session
3. 由于session存放在服务器上，要减轻服务器压力应该使用cookie
4. 单个cookie大小限制为4k，一个站点最多保存20个cookie