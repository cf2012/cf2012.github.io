# Linux 随机端口

启动`hbase`遇到一个问题: 端口被占用.

使用 `ss -nltp | grep 端口` 没有查询到记录.

使用`ss -anp | grep 端口` 找到了对应的进程ID.

问题产生原因:

程序A使用`socket`连接程序S. 此时A需要获得一个本地端口. A有两种方式获取端口:

1. 从操作系统一个随机端口
2. `bind`一个本地端口

操作系统提供的随机端口范围可以通过 `cat /proc/sys/net/ipv4/ip_local_port_range`查看.

该问题产生原因: `Hbase`需要绑定端口正好处于系统提供的随机端口范围内.

## 解决

先修改系统随机端口范围, 避开`hbase`需要绑定的端口. 然后重启占用`hbase`端口的程序.

## 修改系统随机端口返回

打开文件 `/etc/sysctl.conf`. 追加:

```sh
# 32768 开始. 59000 结束
net.ipv4.ip_local_port_range = 32768 59000
```

然后重新加载

```sh
sysctl -p /etc/sysctl.conf
```

验证

```sh
cat /proc/sys/net/ipv4/ip_local_port_range
```
