---
title: "SpringMVC请求乱码解决方案"
date: 2017-04-21 17:27:10 +0800
categories: [Java]
tags: [Tomcat]
---
先在web.xml设置字符编码utf-8

```xml
<filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

如果无法解决接收参数乱码

设置Tomcat配置 **URIEconding="UTF-8"**
编辑文件 /conf/server.xml
```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8" />
```
