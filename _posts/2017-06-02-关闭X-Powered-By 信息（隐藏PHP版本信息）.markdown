---
title: "关闭X-Powered-By 信息（隐藏PHP版本信息）"
date: 2017-06-02 14:28:31 +0800
categories: [Linux]
tags: [PHP]
---
![](http://www.zpq.me/uploads/img/2017-06-02/149638460470653.png)
会暴露服务器运行的是php

修改 php.ini 文件 设置 **expose_php = Off**

官方给出的说明
> Decides whether PHP may expose the fact that it is installed on the server
 (e.g. by adding its signature to the Web server header).  It is no security
 threat in any way, but it makes it possible to determine whether you use PHP
on your server or not.

**决定服务器上是否暴露安装有PHP，
（例如:把这些信息加到Web服务器头响应）。这是不安全的。
但能确定你的服务器时候运行着PHP。
意思就是打开的话可以告诉其他人这台服务器可以运行PHP，但不一定安全，可以关掉**

![](http://www.zpq.me/uploads/img/2017-06-02/149638488657769.png)
