# keepalived+haproxy实现双机热备（未完成）

> 上面通过haproxy实现了负载均衡，但是单个haproxy不满足高可用。于是我们采用keepalived+haproxy的方案实现双机热备。

## 1、安装前准备

keepalived通过争抢某一虚拟ip而实现双机热备，这就要求keepalived和haproxy安装在同一主机上，所以我们在haproxy容器内部安装keepalived

* 进入容器

```shell
docker exec -it haproxy bash
```

* 由于haproxy容器是通过ubuntu上构建的，所以需要采用ubuntu安装软件的命令

```shell
apt-get update
apt-get install keepalived
apt-get install vim
```



## 2、启动keepalived

* 编写keppalived的配置文件`/etc/keepalived/keepalived.conf`

```shell
vrrp_instance  VI_1 {
    state  MASTER
    interface  eth0
    virtual_router_id  51
    priority  100
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  123456
    }
    virtual_ipaddress {
        172.18.0.201
    }
}
```

* 启动keepalived

```shell
service keepalived start 
```

* 测试启动是否成功，在docker主机上

```shell
ping 172.18.0.201
```

