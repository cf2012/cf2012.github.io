# Linux 随机端口

启动`hbase`遇到一个问题: 端口被占用.

使用 `ss -nltp | grep 端口` 没有查询到记录.

使用`ss -anp | grep 端口` 找到了对应的进程ID.

问题产生原因:

程序A使用`socket`连接程序S. 此时A需要获得一个本地端口. A有两种方式获取端口:

1. 从操作系统一个随机端口
2. `bind`一个本地端口

而`A`获得的随机端口正好是`hbase`需要的端口, 哈哈.  (当时`hbase`没有启动, 端口为空闲)

## 临时解决方法

先停掉`A`,释放端口. 然后启动`Hbase`. 然后启动`A`. 搞定

## 思考

系统分配的随机端口范围是什么. 如何避免这个问题?

操作系统提供的随机端口范围可以通过 `cat /proc/sys/net/ipv4/ip_local_port_range`查看.

```bash
$ cat /proc/sys/net/ipv4/ip_local_port_range
32768	60999
```

`hbase`使用的端口恰好在这个范围之内. 

避免此问题复发的方法:

1. 调整`hbase`端口. 有点麻烦 :)
2. 调整 `ip_local_port_range`范围. 开发环境, 并发要求低, 选这个

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

## tips

写程序时,需要注意linux端口问题. 程序绑定的端口不要在 `ip_local_port_range`之内. 

