---
title: "Swoole 整合 Laravel"
date: 2018-05-04 15:28:42 +0800
categories: [PHP]
tags: [Laravel, Swoole]
---
## php常见运行方式

1. php + module + apache
2. php + php-fpm + nginx

## 传统运行方式优点

1. 每次都是新的请求，运行完即释放，不占用内存

## 传统运行方式缺点

1. 每次都需要composer 引入文件
2. DB contention 开销大，每次运行都要建立连接和执行查询，大多数性能消耗在连接上

## 基于swoole http 容器

swoole 有一个优点就是他可以常驻内存，不需要反复引用，类似于JAVA里的Spring Boot
DB contention 也可以有连接池不需要每次执行完就断开连接，减少连接次数。

## 整合代码

安装swoole扩展 ...略

**将代码放入laravel 框架下的public目录下**

```php
<?php
/**
 * Created by PhpStorm.
 * User: zpq
 * Date: 18-5-2
 * Time: 上午8:23
 */


use Symfony\Component\HttpFoundation\ParameterBag;

$server = new Swoole\Http\Server('0.0.0.0',8000);
$app = null;

$server->set([
    'http_parse_post' => true,
    'document_root' => __DIR__,
    'enable_static_handler' => true,
]);

$server->on('WorkerStart',function () {

    global $app;
    require __DIR__.'/../vendor/autoload.php';
    $app = require_once __DIR__.'/../bootstrap/app.php';
});

$server->on('Request',function (\Swoole\Http\Request $sRequest ,\Swoole\Http\Response $sResponse) {

    global $app;
    $_GET = $sRequest->get??[];
    $_POST = $sRequest->post?? [];
    $_COOKIE = $sRequest->cookie ??[];
    $_FILES = $sRequest->files ?? [];
    foreach ($sRequest->server as $key => $value) {

        $_SERVER[strtoupper($key)] = $value;
    }
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    $request = Illuminate\Http\Request::capture();
    foreach ($sRequest->header as $key => $value) {

        $request->headers->set($key,explode(';',$value));
    }
    if ($request->isJson() && $sRequest->rawContent()) {

        $request->setJson(new ParameterBag(json_decode($sRequest->rawContent(),true)));
    }
    $response = $kernel->handle($request);
    foreach ($response->headers->allPreserveCaseWithoutCookies() as $key => $item) {

        $value = implode(";",$item);
        $sResponse->header($key,$value);
    }
    foreach ($response->headers->getCookies() as $item) {

        $sResponse->cookie($item->getName(),$item->getValue(),$item->getExpiresTime(),$item->getPath(),
            $item->getDomain(),$item->isSecure(),$item->isHttpOnly());
    }
    $sResponse->status($response->getStatusCode());
    $sResponse->write($response->getContent());

    $kernel->terminate($request,$response);
});

$server->start();
```
进入*public*目录 执行`php start.php`即可运行。更多server设置属性查看swoole文档

**整合好处 laravel 代码不用变**
