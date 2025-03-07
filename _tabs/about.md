---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---

![](https://github-readme-stats.vercel.app/api?username=zpqsunny&hide_title=true&hide_border=true&show_icons=true&include_all_commits=true&line_height=21&bg_color=0,EC6C6C,FFD479,FFFC79,73FA79&theme=graywhite&locale=en)

![](https://github-readme-stats.vercel.app/api/top-langs/?username=zpqsunny&hide_title=true&hide_border=true&layout=compact&bg_color=0,73FA79,73FDFF,D783FF&theme=graywhite&locale=en)


# 联系方式

- 手机：MTM5MTUzMzY3MjE=
- Email：zpqsunny@gmail.com
- QQ：452617435

---

# 个人信息

- 郑培钦/男/1993
- 本科/东北大学计算机系
- 工作年限：11年
- 技术博客：[https://www.zpq.me](https://www.zpq.me)
- Github：[https://github.com/zpqsunny](https://github.com/zpqsunny)
- 期望职位：JAVA程序员/全栈工程师/技术负责人
- 期望城市：无锡

---

# 工作经历

## 成都任小满科技有限公司 (任我行集团) (2022.8 ~ 至今) - BI研发负责人(BI产品/数据仓库)

### 主要项目
1. 负责BI数据产品研发
2. BI工具产品应用于`海康威视集团` `正新鸡排集团` 可视化数据平台
3. 数据产品应用在集团总部产品销售数据可视化
4. 数据产品移动端/数据模版市场
5. 数仓搭建/任务调度与项目实施

### 技术架构

产品架构: Jdbc + DolphinScheduler + SpringBoot + druid + NFS + 所有分析型/关系型数据库 + React

### 工作职责

1. 负责公司BI工具产品研发(SpringBoot + React) `对标凡软`
2. BI产品平台化开发管理
3. 开发oAuth2认证服务(BI产品整合)
4. 数仓建设(Greenplum/Clickhouse)
5. 数据项目报表实施

### 攻克难关

1. 抽象开发三大数据源,`文件(csv,xls,xlsx)` `http` `jdbc`.
2. 实现功能自定义函数嵌套
3. 实现线性维度和离散维度分组汇总
4. 指标同环比/天气(高德)/国家地图/插件开发
5. 实现不同数据源除法为0时的处理方式及sql方言统一化
6. 增加clickhouse数据源,增加minio文件上传,增加默认主题/自定义主题
7. BI工具平台化`(移动端图表/控制器)`
8. 平台监测钉钉/邮件通知
9. 数仓开发`(ods/dw/dm)`
10. 图表插件式开发
11. 数据模版市场
12. 任务调度`(shell/script/dolphinScheduler)`

### 工作评价

这份工作主要从事大数据BI工具的研发工作，工作中熟练的掌握数仓建设和BI工具的研发
基于前工作的业务经验能够快速的理解业务，分析出业务所需要的维度指标，通过数仓建设数据表，任务调度，最后
通过BI工具进行呈现，从而使数据驱动业务


## 迪锐克斯科技无锡有限公司 (2020.11 ~ 2022.7) - Java开发

### 主要项目
从内部业务开发, ERP, OA, B2B, B2C, 银行, 财务,等模块开发
### 技术架构
分布式架构 + Spring Boot + Spring Cloud + Nacos + Redis + Mybatis + Mysql + ActiveMQ + Vue
### 工作职责
1. 内部前端开发Jquery 转 Vue
2. 内部ERP系统开发
3. 财务系统自动开票
4. 相关报表的统计和导出

### 攻克难关
1. 银行模块通过MQ消息机制进行数据的异步传输, 建立起统一接口和不同银行实现对公付款/对私转账, 实现异步通知结果
   - 中行(Http + XML)
   - 农行(SocketClient)
   - 招商(Http + JSON)
2. 微服务架构系统
3. 天猫,京东,抖音电商接口对接内部B2C系统和内部物流系统`顺丰`,`UPS`, 天猫是MQ异步通知,京东和抖音Http主动获取

### 工作评价
1. 这份工作担任主要技术负责人,带头解决技术问题
2. 提高了团队沟通,协作和管理能力


## 广州有色易购网络有限公司深圳分公司 (2018.03 ~ 2020.10) - Java开发

### 主要项目
该公司主要从事有色金属交易(大宗商品) 期货/现货交易
### 技术架构
Spring Boot + Mysql + Redis  + Mybatis + Vue + Vuex + ElementUI + ECharts
### 工作职责
1. 内部业务系统研发(采购系统,库存系统,供应商系统,利润核算/报表) http://www.zseego.com
2. 数据分析平台(后端:数据爬虫,归档,数据对比,数据公式计算) http://data.zseego.com/data/cu
3. 消息推送(WebSocket长连接)
4. 物流系统(内部业务系统预留出入库接口)
5. 内部业务系统对接财务系统K3
6. 公司官网 http://zhongse.zseego.com/
7. 内部ERP系统开发
8. 内部小程序后台开发

### 攻克难关
1. 公司内部业务系统研发 (内部业务复杂,详细了解业务)
2. 采购系统合同模块开发 (业务合同种样繁多,通过自定义模板和模板自定义字段解决)
3. 数据分析平台
   - 爬虫: 通过Linux的Crontab定时任务处理.爬的网站有:上期所,SMM,上海金属网,国家统计局,海关数据,LME,永安套利,长江有色
   - 数据公式计算:  通过自定义公式进行计算,例 `(#50 + #60) * 2`, 表示的意思是id=50,60的数据集先做和运算最后乘2
4. 物流系统
   - 基于SpringBoot框架实现了公司现货交易线下运输货物实时物流信息查询
   - 每天现货交易巨大后续通ShardingSphere实现了分表查询
5. 公司硬件通讯基于Netty框架TCP长连接方式,采用二进制数据格式进行传输
6. 基于Nacos实现分布式服务架构系统

### 工作评价
1. 在这家公司学到的东西还是很多,业务上的知识,期货上的知识,有了一定见识
2. 技术上也有提高,尤其前端熟练运用了Vue

## 深圳环阳通信息技术有限公司 (2015.10 ~ 2018.02) - Java开发

### 主要项目
该公司主要从事自助售卖
### 技术架构
SpringBoot + Mysql + Hibernate + Redis + Bootstrap3 + Jquery + ECharts
### 工作职责
1. 微信公众号开发
2. 独自开发自助售卖平台和商户后台
3. 独自开发统一支付数据平台(微信,支付宝)
4. 带领内部小团体完成其他项目

### 攻克难关
1. 支付平台的并发 (处理方案基于Spring boot 分布式服务)
2. 线下设备状态信息和数据上传

### 工作评价
1. 这份工作是朋友介绍去做的,该公司主要是生产制造企业想转型.
2. 在这公司工作的同时也继续研究其他框架,阅读源代码.
3. 在做自助售卖平台时,因为当时人手的原因,一个人承担了前端和后端的开发,当时前端还比较 薄弱,在这个项目完成后有了明显的提高,同时看了点Vue
4. 阅读了相关Java的相关书籍 `JavaEE开发的颠覆者 Spring Boot实战` `Java 8编程官方参考教程(第9版)` `阿里巴巴开发手册`


## 无锡金点科技有限公司 (2014.07 ~ 2015.08) - 软件实施

### ERP项目
ERP系统项目的实施与维护，二次开发报表。

---

# 开源项目和作品

## 开源项目

- [Transmission-Web-UI](https://github.com/zpqsunny/Transmission-Web-UI) 为`Transmission`编写的VueUI页面,可用于Chrome,Edge扩展应用商店,用户10k+
- [DHT](https://github.com/zpqsunny/dht) 基于Netty网络框架编写的DHT网络磁力爬虫`Docker`运行

# 技能清单

以下均为我熟练使用的技能

- 操作系统: Linux/Openwrt
- Web开发: Java/PHP
- Web框架: SpringBoot/Netty/Laravel
- 前端框架: Bootstrap/HTML5/ElementUI/Ant Design/Vant
- 前端工具: jQuery/Vue/React
- 数据库相关: MySQL/Redis/MongoDB/Elasticsearch/Postgresql/Clickhouse
- 版本管理、文档和自动化部署工具: Svn/Git/Maven/Docker/K8s/Jenkins
- 单元测试: Unit/SimpleTest/Qunit
- 相关证书: CCNA/网络管理员四级/网络操作员中级

# 致谢
感谢您花时间阅读我的简历，期待能有机会和您共事。
