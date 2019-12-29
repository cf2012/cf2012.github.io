# Linux 配置免密登陆

ssh一般有2种方式认证:

1. 用户名/口令认证. 默认打开
2. 公私钥认证

使用用户名/口令认证安全性低. 有被暴力破解的可能. 公私钥认证安全性高. 方便(不用记录这么多密码了 :-) )

## 公私钥认证设置方式

使用`ssh-keygen`生成自己的公私钥.(推荐设置密码,加密自己的私钥. 长度推荐4096)

```sh
ssh-keygen
```

详细流程参见: [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

生成结果如下:

* ~/.ssh/id_rsa.pub - 公钥
* ~/.ssh/id_rsa - 私钥

复制公钥到要登陆的服务器

```sh
ssh-copy-id USER@hostname
```

> ssh-copy-id 会将公钥写入`USER@hostname`的`~/.ssh/authorized_keys`写. 如果文件不存在,`ssh-copy-id`会自动创建. 
> 其中, `~/.ssh`权限为`0700`, `~/.ssh/authorized_keys`的权限为`0600`. 如果权限不对,免密登陆会失败.

## 其它技巧

可以为ssh连接设置一个别名.方便管理.

比如要登陆`app1`使用命令:

```sh
ssh -p 2222 ubuntu@192.168.10.10
```

设置别名后, 可以直接通过

```sh
ssh app1
```

登陆.

设置方法, 在 ~/.ssh/config 中添加

```ini
# 如果连接空闲. 每隔60秒发送一次数据包. 用于保持连接
ServerAliveInterval 60
TCPKeepAlive no

# 新增的
Host app1
    User ubuntu
    Hostname 192.168.10.10
    Port 2222
```

其它参数:

* IdentityFile ~/path/to/id_file  指定公私钥位置(方便使用多串公私钥)
* ServerAliveInterval X
* ServerAliveCountMax Y
* ServerAliveInterval 60  用于保持连接. 当60秒没有发送数据. 就发一个数据包过去
* TCPKeepAlive no
* Compression  是否压缩
* ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p  通过代理方式连接. 使用场景: 访问内网服务器

## 遇到的错误

设置之后,还是需要输入密码. 反复设置 .ssh 权限之后, 还是不行.

在sshd的日志中看到:

```log
Jul  4 11:13:13 localhost sshd[676]: Authentication refused: bad ownership or modes for directory /home/username
Jul  4 11:13:14 localhost sshd[676]: Accepted password for webapp from ****** port ****** ssh2
Jul  4 11:13:14 localhost sshd[676]: pam_unix(sshd:session): session opened for user webapp by (uid=0)
```

原来是用户的home目录modes不对. 执行 chmod 700 /home/username 之后, 问题解决.

## 参考资料


[Using SSH-based Authentication](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-security#sec-SSH)

[What does 'without password' mean in sshd_config file?](https://askubuntu.com/questions/449364/what-does-without-password-mean-in-sshd-config-file)

[Use Your SSH Config File to Create Aliases for Hosts](https://www.howtogeek.com/75007/stupid-geek-tricks-use-your-ssh-config-file-to-create-aliases-for-hosts/)

[让你的SSH通过HTTP代理或者SOCKS5代理](https://kanda.me/2019/07/01/ssh-over-http-or-socks/)

## tips

1. 推荐用户使用公私钥认证登陆系统.
2. 推荐为连接创建一个别名`alias`

相关配置:

```ini
# 允许 root 登陆, 但不能使用密码认证方式登陆
# 如果不能禁止其它用户使用密码登陆, 也可以单独禁止root用户使用密码登陆
PermitRootLogin without-password

# 禁用密码认证
PasswordAuthentication no

```
