---
title: "12306 抢票脚本 基于laravel console"
date: 2018-01-18 17:37:19 +0800
categories: [PHP]
tags: [Laravel]
---
### 主要接口

1. **POST** [https://kyfw.12306.cn/otn/login/checkUser](https://kyfw.12306.cn/otn/login/checkUser) 验证用户是否登录
2. **GET** [https://kyfw.12306.cn/otn/login/init](https://kyfw.12306.cn/otn/login/init) 登录页面初始化
3. **GET** [https://kyfw.12306.cn/passport/captcha/captcha-image?login_site=E&module=login&rand=sjrand&0.123456789](https://kyfw.12306.cn/passport/captcha/captcha-image?login_site=E&module=login&rand=sjrand&0.123456789) 获取验证码图像接口 _末尾是随机数_
4. **POST** [https://kyfw.12306.cn/passport/captcha/captcha-check](https://kyfw.12306.cn/passport/captcha/captcha-check) 验证码验证
5. **POST** [https://kyfw.12306.cn/passport/web/login](https://kyfw.12306.cn/passport/web/login) 登录请求
6. **POST** [https://kyfw.12306.cn/passport/web/auth/uamtk](https://kyfw.12306.cn/passport/web/auth/uamtk) 获取uamtk _我也不知道是什么玩意_
7. **POST** [https://kyfw.12306.cn/otn/uamauthclient](https://kyfw.12306.cn/otn/uamauthclient) 最后登录成功
8. **POST** [https://kyfw.12306.cn/otn/passengers/init](https://kyfw.12306.cn/otn/passengers/init) 获取乘车人 _其实乘车人可以在请求`提交订单`接口时通过html 正则匹配可以获取，作为抢票工具来说，当然先确定好，后面就只顾抢票就行了。_
9. **GET** [https://kyfw.12306.cn/otn/leftTicket/query](https://kyfw.12306.cn/otn/leftTicket/queryZ) 车次查询
10. **POST** [https://kyfw.12306.cn/otn/leftTicket/submitOrderRequest](https://kyfw.12306.cn/otn/leftTicket/submitOrderRequest) _请求`提交订单`_
11. **POST** [https://kyfw.12306.cn/otn/confirmPassenger/initDc](https://kyfw.12306.cn/otn/confirmPassenger/initDc) _请求订单初始化_
12. **POST** [https://kyfw.12306.cn/otn/confirmPassenger/checkOrderInfo](https://kyfw.12306.cn/otn/confirmPassenger/checkOrderInfo) _请求`验证订单信息`_
13. **POST**[https://kyfw.12306.cn/otn/confirmPassenger/confirmSingleForQueue](https://kyfw.12306.cn/otn/confirmPassenger/confirmSingleForQueue) _确认`订单信息`_

### 注意事项

在调用12306接口时，最好把每次请求所返回的`cookie` 都保存下来，以便下次调用其他接口是顺带上。PHP 建议保存全局`array`、 Java 建议保存一个`Map` 做好每当有新值时做好新增和替换。在请求每一个接口时 要判断body是否未空字符串，很多时候请求返回为空需要重复请求直到有正确返回。

### 接口描述

1. 验证用户是否登录

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| _json_att | 空字符 |

返回：

``` json
{  
    "validateMessagesShowId": "_validatorMessage",  
    "status": true,  
    "httpstatus": 200,  
    "data": {  
        "flag": false  
    },  
    "messages": [],  
    "validateMessages": {}  
}  

```

2. 登录页面初始化

> 这个没什么好说的相当于打开一个页面 **记得保存cookie**

3. 获取验证码图像接口  
_body 返回图像的二进制流，注意： 头部有`Set Cookie : _passport_session=abc;_passport_ct=abc` 建议将body的内容md5做一个验证码库，以后脚本就可以根据验证码库实现自动登录_
4. 验证码验证

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| answer | 图片答案坐标值 |
| login_site | E |
| rand | sjrand |

**当验证码验证失败时，请在全局cookie 中删除两个KEY `_passport_session,_passport_session`，这样才能保证下次获取的验证码图像是有效的**

返回：

``` json
{"result_message":"验证码校验成功","result_code":"4"}  

```

5. 登录请求

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| username | 用户名 |
| password | 密码 |
| appid | otn |

返回：

``` json
{"result_message":"登录成功","result_code":0,"uamtk":"hNFI2iowH46SkgEOdsC1jPSm5MeM50S1wVknuZWOCs4511110"}  

```

6. 获取uamtk

请求：

**将上面返回的uamtk设置到全局`Cookie`中 Set Cookie: uamtk=hNFI2iowH46SkgEOdsC1jPSm5MeM50S1wVknuZWOCs4511110**

| 请求参数 | 请求值 |
| ---- | --- |
| appid | otn |

返回：

``` json
{"result_message":"验证通过","result_code":0,"apptk":null,"newapptk":"1D2_g7EUpnq2hGh6OCxABPYzIkRHuXEHjaLQCIoWh5Erw1110"}  

```

7. 请求最后登录成功

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| tk | 1D2_g7EUpnq2hGh6OCxABPYzIkRHuXEHjaLQCIoWh5Erw1110 |

返回：

``` json
{"apptk":"1D2_g7EUpnq2hGh6OCxABPYzIkRHuXEHjaLQCIoWh5Erw1110","result_code":0,"result_message":"验证通过","username":"某某某"}  

```

8. 获取乘车人

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| \_json\_att | 空字符 |

返回 html：

``` php
//正则匹配出乘车人信息  
preg_match('/passengers=\[.*\];/', $body, $mth);  
$passengerObj = str_replace("passengers=", '', $mth[0]);  
$passengerObj = str_replace(";", '', $passengerObj);  
$passengerJsonStr = str_replace("'", '"', $passengerObj);  
$passengerJson = json_decode($passengerJsonStr, true);  

```

9. 车次查询

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| leftTicketDTO.train_date | 乘车日期`2018-01-01` 格式 |
| leftTicketDTO.from_station | 从哪去 [**对应车站代码**](#%E5%85%B6%E4%BB%96) |
| leftTicketDTO.to_station | 到哪里 [**对应车站代码**](#%E5%85%B6%E4%BB%96) |
| purpose_codes | ADULT |

返回：

``` json
{"data":  
    {"flag":"1",  
     "map":{"BJQ":"深圳东","OSQ":"深圳西","SZQ":"深圳","UNG":"龙南"},  
     "result":[  
"%2Fomy92SddEWJNSkeEqDWLNcqHjN%2F2yds3mb6w6oK4txUa51Av3QrpNUfdUhkrG%2Fk1Y%2FMMfL2X6ap%0AjTJH%2B2cjxLktehVEtsDOBc7F9bdCS1KUzb8Yhasipp1svkaosOo147jsFB9sXwLPl1WmgmzZs7hk%0AfpBRxtnhXzXHvih6H9sN%2Fn5I7%2FJN8blPFpiBXWzMKDPUzQqb4kfBNHlGh%2B%2FVAuwu8pTs85C4oXZ2%0AXGrtpRpO0wNWoHbb5BxmwOs9WEpxyKekxw%3D%3D|预订|240000G1010D|G101|VNP|AOH|VNP|AOH|06:43|12:39|05:56|Y|zRcaqkbreGzvNhhf2Y15yLdPELNQWpW%2FtDFmtCg8bPhFRRSn|20180210|3|P2|01|11|1|0|||||||||||有|有|2||O0M090|OM9|0"]},"httpstatus":200,"messages":"","status":true}  

```

``` php
//result 结果送上 通过 `|` 分隔字符串  
[  
	'cc' => $d[3],  //车次  
    'cf' => $d[8],  //出发时间  
    'dd' => $d[9],  //到达时间  
    'ls' => $d[10], //历时  
    'swz' => $d[32], //商务座/特等座 ok  
    'ydz' => $d[31], //一等座 ok  
    'edz' => $d[30], //二等座 ok  
    'gjrw' => $d[21], //高级软卧 21  
    'rw' => $d[23], //软卧 ok  
    'dw' => $d[33], //动卧 ok  
    'yw' => $d[28], //硬卧 ok  
    'rz' => $d[24], //软座 ok  
    'yz' => $d[29], //硬座 ok  
    'wz' => $d[26], //无座 ok  
    'qt' => $d[22], //其他 ok  
    'secretStr' => urldecode($d[0])  
]  

```

10. 请求提交订单

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| secretStr | 上面php的secretStr |
| train_date | 去程日期 |
| back\_train\_date | 反程日期 |
| tour_flag | `dc` 单程 `wc` 往返 |
| purpose_codes | ADULT |
| query\_from\_station\_name | `始发站名称` |
| query\_to\_station\_name | `到达站名称` |
| undefined | 空字符串 |

返回：

``` json
{"validateMessagesShowId":"_validatorMessage","status":true,"httpstatus":200,"data":"N","messages":[],"validateMessages":{}}  

```

11. 请求订单初始化

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| \_json\_att | 空字符串 |

返回html：

``` php
//正则匹配需要的值： globalRepeatSubmitToken，ticketInfoForPassengerForm  

preg_match("/globalRepeatSubmitToken\h=\h\'.*\'\;/", $body, $ma);  
$st = str_replace("globalRepeatSubmitToken = '", '', $ma[0]);  
$repeatSubmitToken = str_replace("';", '', $st);  

preg_match("/ticketInfoForPassengerForm=.*\}\;/", $body, $ma2);  
//是个json  
$st1 = str_replace("ticketInfoForPassengerForm=", '', $ma2[0]);  
$st2 = str_replace(";", '', $st1);  
$ticketInfoForPassengerForm = str_replace("'", '"', $st2);  
$ticketInfoForPassengerForm = json_decode($ticketInfoForPassengerForm, true);  

```

12. 验证订单信息

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| cancel_flag | 2 |
| bed\_level\_order\_num | 000000000000000000000000000000 |
| passengerTicketStr | 座位类型，0，车票类型，姓名，身份正号，电话，N（多个的话，以逗号分隔） |
| oldPassengerStr | 姓名，证件类别，证件号码，用户类型 |
| tour_flag | `dc` 单程 `wc` 往返 |
| randCode | 空字符串 |
| whatsSelect | 1 |
| \_json\_att | 空字符串 |
| REPEAT\_SUBMIT\_TOKEN | 上面php repeatSubmitToken |

返回：

``` json
{"validateMessagesShowId":"_validatorMessage","status":true,"httpstatus":200,"data":{"ifShowPassCode":"N","canChooseBeds":"N","canChooseSeats":"Y","choose_Seats":"OM","isCanChooseMid":"N","ifShowPassCodeTime":"731","submitStatus":true,"smokeStr":""},"messages":[],"validateMessages":{}}  

```

13. 确认订单信息

请求：

| 请求参数 | 请求值 |
| ---- | --- |
| passengerTicketStr | 同上 |
| oldPassengerStr | 同上 |
| randCode | 空字符串 |
| purpose_codes | 00 |
| key\_check\_isChange | 上面 php $ticketInfoForPassengerForm\['key\_check\_isChange'\] |
| leftTicketStr | 上面 php $ticketInfoForPassengerForm['leftTicketStr'] |
| train_location | 上面 php $ticketInfoForPassengerForm['train_location'] |
| choose_seats | 空字符串 |
| seatDetailType | 000 |
| whatsSelect | 1 |
| roomType | 00 |
| dwAll | N |
| \_json\_att | 空字符串 |

返回：

``` json
{"validateMessagesShowId":"_validatorMessage","status":true,"httpstatus":200,"data":{"submitStatus":true},"messages":[],"validateMessages":{}}  

```

### 其他

* 对应车站代码 [车站js获取](https://kyfw.12306.cn/otn/resources/js/framework/station_name.js?station_version=1.9044) 大写的英文字母就是车站代码
* 自己写的抢票demo php :fa-github: [github](https://github.com/zpqsunny/ticket)
