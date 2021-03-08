---
title: 在 Centos 7 上搭建 strongSwan VPN IKEv2 整合 FreeRadius 用户认证系统
date: 2019-06-26 17:47:37 +0800
categories: [Linux]
tags: [VPN, strongSwan, FreeRadius]
---
## 第一步

### 安装strongSwan

```shell
yum -y install epel-release
yum -y install strongswan openssl
```

```shell
strongswan pki --gen --type rsa --size 4096 --outform pem > ca.key.pem
strongswan pki --self --in ca.key.pem --dn "C=CN, O=Linux strongSwan, CN=VPN CA" --ca --lifetime 3650  --type rsa --outform pem > ca.cert.pem
```

* –self 表示自签证书
* –in 是输入的私钥
* –dn 是判别名
* –ca 表示生成 CA 根证书
* –lifetime 为有效期, 单位是天
* C 表示国家名，同样还有 ST 州/省名，L 地区名，STREET（全大写） 街道名
* O 组织名称
* CN 友好显示的通用名

服务器外网IP : 10.1.1.2

```shell
strongswan pki --gen --type rsa --size 4096 --outform pem > server.key.pem
strongswan pki --pub --in server.key.pem --outform pem > server.pub.pem
strongswan pki --pub --in server.key.pem | strongswan pki --issue --lifetime 3650 --cacert ca.cert.pem \
--cakey ca.key.pem --dn "C=CN, O=Linux strongSwan, CN=10.1.1.2" \
--san="10.1.1.2" --san="10.1.1.2" --flag serverAuth --flag ikeIntermediate \
--outform pem > server.cert.pem
```

* –issue, –cacert 和 –cakey 就是表明要用刚才自签的 CA 证书来签这个服务器证书。
* –dn, –san，–flag 是一些客户端方面的特殊要求：

iOS 客户端要求 CN 也就是通用名必须是你的服务器的 URL 或 IP 地址;  
Windows 7 不但要求了上面，还要求必须显式说明这个服务器证书的用途（用于与服务器进行认证），–flag serverAuth;  
非 iOS 的 Mac OS X 要求了“IP 安全网络密钥互换居间（IP Security IKE Intermediate）”这种增强型密钥用法（EKU），–flag ikdeIntermediate;  
Android 和 iOS 都要求服务器别名（serverAltName）就是服务器的 URL 或 IP 地址，–san。

复制证书到指定目录

```shell
cp -r ca.key.pem /etc/strongswan/ipsec.d/private/
cp -r ca.cert.pem /etc/strongswan/ipsec.d/cacerts/
cp -r server.cert.pem /etc/strongswan/ipsec.d/certs/
cp -r server.pub.pem /etc/strongswan/ipsec.d/certs/
cp -r server.key.pem /etc/strongswan/ipsec.d/private/
```

配置VPN *这里只配置了IKEV2-mschapv2认证方式* [IpsecConf](https://wiki.strongswan.org/projects/strongswan/wiki/IpsecConf)

```shell
# vim /etc/strongswan/ipsec.conf
```

```shell
config setup
        # strictcrlpolicy=yes    # 是否严格执行证书吊销规则
        uniqueids = never        # 如果同一个用户在不同的设备上重复登录,yes 断开旧连接,创建新连接;no 保持旧连接,并发送通知; never 同 no, 但不发送通知.
conn %default

        compress = yes                  # 是否启用压缩, yes 表示如果支持压缩会启用
        dpdaction = clear                # 当意外断开后尝试的操作
        dpddelay = 30s                  # dpd时间间隔
        inactivity = 300s               # 闲置时长,超过后断开连接
        left = %any                     # 服务器端标识，可以是魔术字 %any，表示从本地ip地址表中取
        leftid = 10.1.1.2               # 服务器端ID标识
        leftsubnet = 0.0.0.0/0          # 服务器端虚拟IP，0.0.0.0/0表示通配
        leftcert = server.cert.pem      # 服务器端证书
        right=%any                      # 客户端标识，%any表示任意
        rightsourceip=192.168.99.0/24       # 客户端IP地址分配范围

conn IKEv2-EAP
        keyexchange = ikev2             # 使用 IKEv2
        leftca = "C=CN, O=Linux strongSwan, CN=VPN CA" # 服务器端根证书DN名称
        leftsendcert = always           # 是否发送服务器证书到客户端
        rightsendcert = never           # 客户端不发送证书
        left=%any                       # 服务器端标识,%any表示任意
        leftauth=pubkey                 # 服务器校验方式，使用证书
        rightauth=eap-mschapv2          #KEv2 EAP(Username/Password)
        eap_identity = %any             # 指定客户端eap id
        rekey = no                      # 不自动重置密钥
        auto = add                      # 当服务启动时, 应该如何处理这个连接项,add 添加到连接表中
        ike = aes256-aes128-3des-sha1-modp1024!
                                        # 密钥交换协议加密算法列表，可以包括多个算法和协议。
                                        # 5.0.2增加了配置与完整性保护定义的PRF算法不同的PRF算法的能力。
        esp = aes256-3des-sha256-sha1!  
                                        # 数据传输协议加密算法列表,对于IKEv2，可以在包含相同类型的多个算法（由-分隔）。
                                        # IKEv1仅包含协议中的第一个算法。只能使用ah或esp关键字，不支持AH+ESP。
```

配置strongSwan参数

```shell
# vim /etc/strongswan/strongswan.conf
```

```shell
charon {
        load_modular = yes
        plugins {
                include strongswan.d/charon/*.conf
        }
        dns1 = 8.8.8.8          #配置客户端使用的DNS
        dns2 = 8.8.4.4

}
include strongswan.d/*.conf
```

配置VPN账号 [IpsecSecrets](https://wiki.strongswan.org/projects/strongswan/wiki/IpsecSecrets)

```shell
vim /etc/strongswan/ipsec.secrets
```

用户名: user  
密码: password

```shell
# ipsec.secrets - strongSwan IPsec secrets file
# 使用证书验证时的服务器端私钥
# 格式 : RSA <private key file> [ <passphrase> | %prompt ]
: RSA server.key.pem
#EAP 方式, 格式同 psk 相同
user %any : EAP "password"
```

内核转发

```shell
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1" >> /etc/sysctl.conf
sysctl -p
```

配置firewalld防火墙 *端口允许和ip伪装*

```shell
firewall-cmd --permanent --add-service="ipsec"
firewall-cmd --permanent --add-port=500/udp
firewall-cmd --permanent --add-port=4500/udp
firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
```

```shell
systemctl start strongswan
systemctl enable strongswan
```

## 第二步

### 客户端测试

#### IOS

1. 先导入 CA 证书 将之前创建的`ca.cert.pem`文件放在web中，使用iOS自带的浏览器Safari访问ca.cert.pem,然后进行安装。
2. 使用 IKEv2 + EAP 认证找到手机上 “设置->VPN->添加配置”, 选IKEv2

描 述：随便填  
服 务 器：10.1.1.2 #填url或ip  
远 程 ID：10.1.1.2 #填url或ip  
用户鉴定：用户名  
用 户 名：username #EAP 项用户名  
密 码：password #EAP 项密码

#### Windows10

1. 将 CA 根证书 ca.cert.pem 重命名为 ca.cert.crt
2. 按win+r 运行mmc;
3. 文件>添加删除管理单元；
4. 在可用的管理单元中选择“证书”，点击添加 > 选择本地计算机用户> 确定； 在控制节点中展开证书 > 受信任的证书颁发机构 > 右击所有任务 > 导入。

“控制面板”-“网络和共享中心”-“设置新的连接或网络”-“连接到工作区”-“使用我的 Internet 连接”  
Internet 地址写服务器 IP 或 URL。  
描述随便写。  
用户名密码写之前配置的 EAP 的那个。  
确定  
转到 控制面板网络和 Internet网络连接  
在新建的 VPN 连接上右键属性然后切换到“安全”选项卡  
VPN 类型选 IKEv2  
数据加密选“需要加密”  
身份认证，使用 EAP 认证选择“Microsoft:安全密码(EAP-MSCHAP v2)”;  
再切换到 “网络” 选项卡, 双击 “Internet 协议版本 4” 以打开属性窗口,  
点击 “高级” 按钮, 勾选 “在远程网络上使用默认网关”, 确定退出。

**测试是否无误 没有问题再进行第二步**

## 第二步

### 安装freeradius

```shell
yum install -y freeradius freeradius-mysql # 这里用了mysql数据库如果需要sqlite 换成 freeradius-sqlite
```

```shell
vim /etc/raddb/clients.conf
client localhost {
        ipaddr = 127.0.0.1
        secret = 123456789 //修改 secret 的密码
```

设置用户名和密码 testing password

```shell
vim /etc/raddb/users
testing Cleartext-Password := "password"
```

尝试运行服务在Debug模式

```shell
radiusd -X
Read to process requests. # 如果看到这行说明能成功运行
```

导入radius 所需要的数据库脚本

```shell
mysql -uroot -p
  CREATE DATABASE radius;
  exit
mysql -uroot -p radius < /etc/raddb/mods-config/sql/main/mysql/schema.sql
mysql -uroot -p radius < /etc/raddb/mods-config/sql/main/mysql/setup.sql # 这个可执行也可不执行 非必需 该脚本创建一个mysql用户
```

配置FreeRadius 的SQL

```shell
cd /etc/raddb/mods-enabled
ln -s ../mods-available/sql sql
vim sql
sql {
    driver = "rlm_sql_mysql"
    dialect = "mysql"
    # Connection info:
    #
    server = "localhost"
    port = 3306
    login = "root"
    password = "root"
    # Database table configuration for everything except Oracle
    radius_db = "radius"
}
```

编辑`/etc/raddb/sites-available/default`取消`authorize {}`部分中包含sql的注释行。即 `-sql` 修改为 `sql`  
此外，编辑`/etc/raddb/sites-available/inner-tunnel`取消`authorize {}`下包含sql的注释行。即 `-sql` 修改为 `sql`

新增用户名/密码 test/testpassword

```shell
mysql -u radius -p
use radius;
insert into radcheck (username,attribute,op,value) values("test", "Cleartext-Password", ":=", "testpassword");
```

再次执行 `radiusd -X` 查看是否有问题

```shell
systemctl start radiusd
```

## 第三步

### 整合

**请确保以上都正确执行没有错误**

```shell
vim /etc/strongswan/strongswan.d/charon/eap-radius.conf
eap-radius {
    accounting = yes
    load = yes
    port = 1812
    secret = 123456789
    server = 127.0.0.1
}
```

strongSwan 修改认证方式

```shell
vim /etc/strongswan/ipsec.conf
conn IKEv2-EAP {
    rightauth=eap-radius # eap-mschapv2 修改成 eap-radius
}
vim /etc/strongswan/ipsec.secrets
#user %any : EAP "password" #该行注释掉用radius认证
```

```shell
systemctl restart strongswan
systemctl restart radiusd
```

最后用 test testpassword 进行账户登陆

# 免CA证书实现登录

免CA证书的主要原理还是基于信任,上面的手动安装CA证书就要设置信任,所以只要我们有一些机构签发的证书,那就可以试用,这里主要使用**Let'sencrypt** 使用这个需要每三个月需要一次续签

```shell
yum -y install certbot
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
certbot certonly --rsa-key-size 4096 --standalone --agree-tos --no-eff-email --email hakase@gmail.com -d 域名
```
证书路径 **/etc/letsencrypt/live/域名**
```shell
ln -s /etc/letsencrypt/live/域名/fullchain.pem /etc/strongswan/ipsec.d/certs/
ln -s /etc/letsencrypt/live/域名/privkey.pem /etc/strongswan/ipsec.d/private/
ln -s /etc/letsencrypt/live/域名/chain.pem /etc/strongswan/ipsec.d/cacerts/
```
使用链接的好处就是下次更新时不需要复制文件
1. 修改**ipsec.conf** leftcert = fullchain.pem
2. 修改**ipsec.conf** 删除**leftca**
```shell
vim ipsec.secrets
//上面的 : RSA server.key.pem  修改成以下并保存
: RSA "privkey.pem"
```
> [https://www.jianshu.com/p/c63e3e163075](https://www.jianshu.com/p/c63e3e163075)
> [http://wiki.freeradius.org/guide/Basic-configuration-HOWTO](http://wiki.freeradius.org/guide/Basic-configuration-HOWTO)  
> [http://wiki.freeradius.org/guide/SQL-HOWTO-for-freeradius-3.x-on-Debian-Ubuntu](http://wiki.freeradius.org/guide/SQL-HOWTO-for-freeradius-3.x-on-Debian-Ubuntu)
> [how-to-setup-ikev2-vpn-using-strongswan-and-letsencrypt-on-centos-7](https://www.howtoforge.com/tutorial/how-to-setup-ikev2-vpn-using-strongswan-and-letsencrypt-on-centos-7/)
