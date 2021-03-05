---
title: "Openwrt安装composer的一些问题"
date: 2020-06-24 09:23:54 +0800
categories: [Linux]
tags: [Openwrt]
---
openwrt 安装composer遇到的问题

``composer Failed to decode zlib stream``

可能存在的问题是

1. 依赖没有安装
2. 网络下载问题 **(大多数问题)**

```shell
opkg update
opkg install zlib
```

composer 运行出现 ``Timezone database is corrupt -this should *never* happen ``

解决方法

1. 设置系统-系统-时区为自己的地区，例如Asia/ShangHai
2. 安装软件包 zoneinfo-asia，其他地区请将asia改一下
