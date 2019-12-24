# 安装Linux系统注意点

## 安装前

需要确定:

1. 有没有`ntp`?
2. 有可以连的`yum`源吗?
3. 磁盘规划. 如何划分

## 安装

推荐最小化安装

## 安装后

1. 检查`/etc/hosts`. `hostname`是否配置在里面. `localhost`是否配置在里面

检查方法:

```sh
# 查看主机名对应的IP
hostname -i

# 期望: ping到127.0.0.1 的地址
ping localhost
```

原因: 有些程序程序需要通过`hostname`查询本机`ip`.  有些程序需要绑定`localhost` (比如: `pyspark`)
