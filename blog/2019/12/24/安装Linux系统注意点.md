# 安装Linux系统注意点

## 安装前

需要确定:

1. 有没有`ntp`?
2. 有可以连的`yum`源吗?
3. 磁盘规划. 如何划分

## 安装

推荐最小化安装

## 安装后

### 1. 检查`/etc/hosts`. `hostname`是否配置在里面. `localhost`是否配置在里面

检查方法:

```sh
# 查看主机名对应的IP
hostname -i

# 期望: ping到127.0.0.1 的地址
ping localhost
```

原因: 有些程序程序需要通过`hostname`查询本机`ip`.  有些程序需要绑定`localhost` (比如: `pyspark`)

### 2. 是否要修改最大连接数?

​Maximum Open Files Requirements

### 3. 主机名`FQDN`格式

`hostname`推荐采用[`FQDN`格式](https://en.wikipedia.org/wiki/Fully_qualified_domain_name). 主机名类似:

> hd01.example.com
> hd02.example.com

`/etc/hosts`中配置为:

```sh
# localhost. FQDN  格式
127.0.0.1 localhost.localdomain VM-0-11-ubuntu
127.0.0.1 localhost

# FQDN
192.168.1.10 hd01.example.com  hd01
```

测试方法:

```bash
# Display the FQDN
hostname -f
```

参考:

1. [完整網域名稱](https://zh.wikipedia.org/wiki/%E5%AE%8C%E6%95%B4%E7%B6%B2%E5%9F%9F%E5%90%8D%E7%A8%B1)
2. `man hostname`
