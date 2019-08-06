# linu虚拟机固定ip

* 1

```shell
cd /etc/sysconfig/network-scripts/
```

* 2

```shell
vi ifcfg-ens33
```

* 3

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static			#修改
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=413eccb1-47c9-46a0-801b-ec79cefd6534
DEVICE=ens33
ONBOOT=yes					#修改

####添加
IPADDR=192.168.36.111
GATEWAY=192.168.36.2
DNS1=114.114.114.114
NETMASK=255.255.255.0
```

虚拟机网卡和主机虚拟网卡（192.168.36.1）需要在同一网段才能通信。