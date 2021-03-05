---
title: "如何正确的使用Google Cloud 计算引擎"
date: 2017-06-20 10:34:22 +0800
categories: [Linux]
tags: [Google, Cloud]
---
### 必备条件
1. 能翻墙
2. 需要一张外币信用卡 类似`VISA`卡
3. Google账户

注册Google账户登录进去，然后绑定 VISA 卡 绑定的时候会收取1美元验证银行卡 只要验证通过后会退还。

进入 [Google Cloud](console.cloud.google.com) `菜单栏` -> `结算` -> `概览`

如果是第一次使用会赠送300美元 有效期一年

### 创建ssh密钥
Google Cloud 是使用ssh 证书认证进行登录的，所以需要先创建密钥。

`菜单栏` -> `计算引擎` -> `元数据` -> `ssh密钥` -> `修改`

密钥如何生成？这里有个工具 [puttygen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
运行 `puttygen.exe` 点击 `Generat` 等待一会时间后会生成密钥

![](http://www.zpq.me/uploads/img/2017-06-20/149792357673616.png)

然后将key里的内容复制到`ssh密钥`里的文本框中，**注意最后 Key comment后面的字符串不要复制进去** `Key passphrase` 输入两次密码，最后点击 `Save private key` 保存私钥，`ssh密钥`后再追加用户邮箱**即登录账户**

大概最后`ssh密钥`内容如下。
```
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAgZ4ktSCVFiTWImz1Q0uddhNi1qygf6lukKh5E9fC7+EFXRkr9uZ2ws7A2KXuP8xzdpx/yrTSU5kPy852gzmu/Z6vUeYkDqFz0kqosJJBDSn68oFT6lEbAc7LexrARyTOK5Zdej+ohI+HvON+x++FlqWqo78kROfp+2reoGmsccqDfcOzJkt4UksE69nJBpxpiALzsRxfUbIq2Ct3JlwfSFxfUNK6VBZQVGqu1zPIAJF69nq9p0yuKl6kOFGGGWyxKNzBp2WQrX8iNRFZN6WXns3DJsri5eXEVQnyHsjitAXj3RMqTMCfCqjhJOfgTef2R5WchbkYCiszi4z7CyfBGw== user@example.com
```
**ssh-rsa 后面有空格 ，user@example.com 前也有空格**
最后保存

### 创建vm实例
`菜单栏` -> `计算引擎` -> `VM实例` -> `创建实例`

创建实例这边就说几个注意点，其他的一般都默认。

**地区** 可以看这两个网址上的地理分布 https://cloud.google.com/compute/docs/regions-zones/regions-zones
https://cloud.google.com/about/locations/

**防火墙** 允许 http https 流量打勾

**ssh密钥** 选项中 不要打勾，打勾了，之前创建的ssh密钥就不能用，上面的密钥是属于**项目密钥**，通用所有实例，打勾了，就要为该实例单独设置ssh密钥。

最后点击创建 :smiley:
创建完后用[Best Trace](http://www.ipip.net/download.html#ip_trace)测试了下，速度完全秒杀国内运营商。
![](http://www.zpq.me/uploads/img/2017-06-20/149792546750037.png)
![](http://www.zpq.me/uploads/img/2017-06-20/149792548821104.png)
### ssh登录
下载并打开[putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
Session 栏中输入 ip地址
Connection - SSH - Auth 选择最开始保存的私钥文件
Open
用户名 就是*user@example.com* 中的**user**

**执行命令修改root 密码！**
```shell
sudo passwd
```
