# linux常用指令

## 1、防火墙状态

查看防火墙状态

```shell
systemctl status firewalld 
firewall-cmd --state
```

开启/关闭防火墙

```shell
systemctl start firewalld
systemctl stop firewalld
```

重启防火墙

```shell
systemctl restart firewalld
```

开启启动防护墙/查看是否开机启动

```shell
systemctl enable firewalld
systemctl is-enabled firewalld
```



## 2、端口情况

开启端口

```shell
firewall-cmd --zone=public --permanent --add-port=8080/tcp
```

重载防火墙

```shell
firewall-cmd --reload
```

删除端口

```shell
firewall-cmd --permanent --remove-port=8080/tcp
```

查看开启的端口

```shell
firewall-cmd --permanent --list-ports
```

查看开启的服务

```shell
firewall-cmd --permanent --list-services
```



## 3、其他常用指令

查看端口监听情况

```shell
netstat -anp | grep 3306
```

查看进程情况

```shell
ps -aux | grep redis
```

新建软连接

```shell
ln -s redis-5.0.5 redis
```

文件重定向

	# >>和>都属于输出重定向：
	# >会覆盖目标的原有内容。当文件存在时会先删除原文件，再重新创建文件，然后把内容写入该文件；否则直接创建文件; 
	# >>会在目标原有内容后追加内容。当文件存在时直接在文件末尾进行内容追加，不会删除原文件；否则直接创建文件。
```shell
cat redis.conf | grep -v "#" | grep -v "^$" > redis-6666.conf
# -v是grep排除的参数
# ^代表行首，$代表行尾。 ^$是空行的意思
```

