---
title: "@FeignClient 注解 fallback不起作用"
date: 2017-11-21 13:55:34 +0800
categories: [Java]
tags: [Spring Cloud]
---
可能是没有声明Feign的 hystrix支持；
在配置文件中设置：
```yaml
feign:
  hystrix:
    enabled: true
```
