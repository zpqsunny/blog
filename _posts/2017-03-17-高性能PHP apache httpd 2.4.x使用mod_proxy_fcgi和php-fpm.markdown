---
title: "高性能PHP apache httpd 2.4.x使用mod_proxy_fcgi和php-fpm"
date: 2017-03-17 15:16:26 +0800
categories: [PHP]
tags: [PHP, Apache]
---
```shell
yum install -y httpd mod_proxy_fcgi
```
编辑http.conf
在虚拟主机配置php-fpm 代理
```shell
ProxyPassMatch  ^/(.*\.php(/.*)?)$  fcgi://127.0.0.1:9000/path/to/your/documentroot/$1
DirectoryIndex /index.php index.php
```
#### ProxyPassMatch
在这种情况下：只有与指定的正则表达式模式匹配的代理内容

#### ^/(.*\.php(/.*)?)$
从文档根开始，匹配以.php结尾的所有内容（带点转义），可选的后跟一个斜杠和任何你喜欢的继续路径（一些应用程序使用这个所谓的PathInfo将参数传递给php脚本。
^（插入符号）和$（美元）符号用于锚定URL的绝对开始和结束，以确保请求中的任何字符都不会转义我们的模式匹配。
嵌套括号使我们能够将整个请求URI（减去前导斜杠）引用为$ 1，同时仍然保持尾随pathinfo可选。
#### fcgi://127.0.0.1:9000
通过mod_proxy_fcgi转发，使用fastCGI协议，到我们的php-fpm守护程序监听的端口。
这确定哪个fastcgi池将服务由此规则代理的请求。

#### /path/to/your/documentroot
重要！ 这必须与您的php文件的真实文件系统位置完全匹配，因为这是php-fpm守护程序将查找它们的位置。
php-fpm只是解释传递给它的php文件; 它不是一个Web服务器，也不了解您的Web服务器的命名空间，虚拟主机布局或别名。
重要！
#### $1
从原始请求扩展到整个请求URI，减去前导斜杠（因为我们已经添加了上面的。）
#### DirectoryIndex /index.php index.php
对/将需要映射到fcgi后端上的资源的请求。 未能解决此问题可能会导致空白响应，通常称为WSOD（死亡白屏），特别是如果仅代理包含php扩展名的请求URI，例如本示例。 处理链将首先将针对/的请求映射到/index.php或相对于当前请求uri的任何其他index.php文件，然后正确地代理到PHP-FPM后端。
#### unix domain socket (UDS) approach
编辑您选择的vhost的配置，并向其添加以下行：
```shell
ProxyPassMatch ^/(.*\.php(/.*)?)$  unix:/path/to/socket.sock|fcgi://127.0.0.1:9000/path/to/your/documentroot/
```

#### unix:/path/to/socket.sock
您的fpm套接字的路径
请注意，使用此方法，捕获的请求URI（$1）不会在路径之后传递

> 原文出自于 [https://wiki.apache.org/httpd/PHP-FPM](https://wiki.apache.org/httpd/PHP-FPM)
