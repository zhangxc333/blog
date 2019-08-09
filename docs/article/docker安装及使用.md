

# docker安装及使用

## 1、安装操作

* 更新yum

```shell
yum -y update
```

* 修改yum源（aliyun）

```shell
https://mirrors.aliyun.com/repo/Centos-7.repo
```

放在/etc/yum.repos.d/目录中，并改名为CentOS-Base.repo

```shell
yum clean all
yum makecache
```

* 安装docker

```shell
yum -y install docker
# 其他常用工具
yum -y install tree net-tools wget
```

* 修改国内下载镜像（DaoCloud）

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

修改docker配置文件

```shell
vi /etc/docker/daemon.json
```

删除该列中的逗号，并保存推出

```shell
["http://f1361db2.m.daocloud.io"],
```

* docker启动/关闭相关操作

```shell
systemctl status docker
systemctl start docker
systemctl stop docker
systemctl restart docker
systemctl enable docker
systemctl is-enable docker
```

## 2、镜像相关操作

```shell
docker iamges
docker search java
docker pull docker.io/java
docker rmi docker.io/java
#保存和加载镜像
docker save docker.io/java > /home/java.tar.gz
docker load > /home/java.tar.gz
```

## 3、容器相关操作

* 运行镜像，启动容器

```shell
docker run -it -p 9000:8080 -p 9001:8085 --name myjava -v /home/project:/soft --privilege docker.io/java bash
##############
1、-it 表示以交互的方式运行镜像，运行之后直接进入到容器内部，同过exit推出容器
   -i: 以交互模式运行容器，-t: 为容器重新分配一个伪输入终端，通常与 -i、-t 同时使用；
   -d 以后台运行的方式
   # 对于以后台启动的容器，可以这样进入e.g. docker exec -it haproxy bash
2、-p 开启端口映射，将主机的端口（这里docker主机是cnetos虚拟机）：容器的端口（docker容器）
3、-v 绑定一个卷，实现文件映射

```



