# linux防火墙使用

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
firewall-cmd --permanent --add-port=8080/tcp
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

