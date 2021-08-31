# 处理 Alibaba Druid 链接池15分钟超时问题

## 问题

一个在线系统遇到一个奇怪的问题.有些请求耗时长达15分钟. 甚至还有些耗时超过30分钟.  通过`APM`工具分析该请求,发现在测试`DB`连接是否有效时会等待15分钟,然后报错:

> PSQLException: An I/O error occured while sending to the backend

## 原因分析

**为什么会报这个错误?**

根据报错信息在[stackoverflow](https://stackoverflow.com/questions/17285340/postgresql-exception-an-i-o-error-occured-while-sending-to-the-backend)上搜索到了原因: 连接长时间未使用已经失效了.  [<sup> 1 <sup/>](#refer-anchor-1)

**连接为什么会失效?**

失效的原因可能的原因为:  连接长时间空闲. 被防火墙或者交换机等网络设备切断了. [<sup>2<sup/>](#refer-anchor-2)

**为什么长时间空闲?**

系统每天请求量太低,一天不超过150笔 😓

程序里有个 配置: `testWhileIdle`, 空闲时会测试连接是否有效. 当连接长时间不使用,会被网络设备切断. 此时程序没有感知到. 

n秒后,用户发过来一个请求. 连接池拿到连接后,测试一下连接是否有效. 该连接已经被切断了,导致连接不到`DB` ,经过15分钟后,被操作系统判定`socket`连接超时. 最终, `jdbc`驱动报错: 

> An I/O error occured while sending to the backend

## 解决方法

设置参数: `validationQueryTimeout` [<sup>3<sup/>](#refer-anchor-3)

参数含义如下: [<sup>5</sup>](#refer-anchor-5)
```txt
validationQueryTimeout:
单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法
```

## 参考资料

<div id="refer-anchor-1"/>
- [1]  [PostgreSQL Exception: “An I/O error occured while sending to the backend”](https://stackoverflow.com/questions/17285340/postgresql-exception-an-i-o-error-occured-while-sending-to-the-backend)
<div id="refer-anchor-2"/>
-  [2] [Re: An I/O error occured while sending to the backend](https://www.postgresql.org/message-id/42540D4D.2000504%40hogranch.com)
<div id="refer-anchor-3"/>
- [3] [DRUID SOCKET TIMEOUT超时15分钟(转载)](https://www.cnblogs.com/yinliang/p/10996210.html)
<div id="refer-anchor-4"/>
- [4] [【转】深入理解JDBC的超时设置](https://www.jianshu.com/p/6d19e0d7f81c)
<div id="refer-anchor-5"/>
- [5] [DruidDataSource配置属性列表](https://github.com/alibaba/druid/wiki/DruidDataSource配置属性列表)

