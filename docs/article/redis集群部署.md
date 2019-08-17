# redis集群部署

> rdis集群中的数据库复制是通过主从同步来实现的，主节点（master）把数据分发给从节点（slava），主从同步的好处在于高可用，redis界定啊有冗余设计。
>
> redis中文网更新，直接看[https://redis.io](https://redis.io/)更好。

## 一、直接在centos7上部署

### 1、根据redis中文网[下载页面](http://www.redis.cn/download.html)，下载、解压、编译Redis。

```shell
# 这里我习惯在/usr目录下进行（/usr目录用于存放安装程序）
$ wget http://download.redis.io/releases/redis-5.0.4.tar.gz
$ tar xzf redis-5.0.4.tar.gz
$ cd redis-5.0.4
$ make
```

**由于rdis是用c++语言编写的，在执行make是如果出现错误，可能是因为没有安装c++ 的编译器：**

```shell
# 1.
yum -y install gcc-c++
# 2.如果执行make指令出错，则删除文件解压后的redis-5.0.4文件，再次解压，并在执行make语句之前，先执行make clean
$ cd redis-5.0.4
$ make clean
$ make
```

### 2、根据redis中文网[Redis 集群教程](http://www.redis.cn/topics/cluster-tutorial.html)，搭建并使用Redis集群。

#### 1、前期准备

* 新建相编译出可执行文件 redis-server ， 并将文件复制到 cluster-test 文件夹行文件

```shell
# 这里配置文件直接存放在/usr/redis-5.0.4/目录下（/etc目录用于存放配置文件）
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002 7003 7004 7005
```

* 在文件夹 7000 至 7005 中， 各创建一个 `redis.conf` 文件， 文件的内容可以使用下面的示例配置文件， 但记得将配置中的端口号从 7000 改为与文件夹名字相同的号码。

```shell
# 是一个最少选项的集群的配置文件:
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
daemonize yes 
```

```shell
# 这里添加了一个后台运行（要不然在每个标签启动一个redis太麻烦了），官方教程中没有
daemonize yes
```

* 将第一步中，编译出的可执行文件 `redis-server` ，复制到 cluster-test 文件夹中。

```shell
# 如果你在src目录下没找到redis-server，也表明第一步安装失败
cp /usr/redis-5.0.4/src/redis-server  /usr/redis-5.0.4/cluster-test
```

* 使用类似以下命令， 在每个标签页中打开一个实例：

```shell
cd 7000
../redis-server ./redis.conf
```

**这一步都是重复操作，可以考虑编写简单的shell脚本**

* 编写`.sh`文件

  * ```shell
    # 这里注意在linux中编写，在window中编写再copy过来执行的时候会出现问题
    [root@vm cluster-test]# vi start_all.sh
    
    cd 7000
    ../redis-server ./redis.conf
    cd ..
    cd 7001
    ../redis-server ./redis.conf
    cd ..
    cd 7002
    ../redis-server ./redis.conf
    cd ..
    cd 7003
    ../redis-server ./redis.conf
    cd ..
    cd 7004
    ../redis-server ./redis.conf
    cd ..
    cd 7005
    ../redis-server ./redis.conf
    cd ..
    ```

* 修改权限，并执行

  * ```shell
    chmod +x start_all.sh
    ./start_all.sh
    ```

#### 2、搭建集群

* 在可执行文件`redis-cli`目录下（也就是/usr/redis-5.0.4/src/）执行下列语句

```shell
# create, since we want to create a new cluster. 
# the option --cluster-replicas 1 means that we want a slave for every master created.
# The other arguments are the list of addresses of the instances I want to use to create the new cluster.
./redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
--cluster-replicas 1
```

* 如果觉得还行，就输入yes

```shell
Can I set the above configuration? (type 'yes' to accept):
#最后看到下面提示表示创建成功
[OK] All 16384 slots covered
```

#### 3、使用redis集群

* 在可执行文件`redis-cli`目录下（也就是/usr/redis-5.0.4/src/）执行下列语句

```shell
# 进入集群中指定的7000端口对应的redis
$ ./redis-cli -c -p 7000
redis 127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
redis 127.0.0.1:7002> set hello world
-> Redirected to slot [866] located at 127.0.0.1:7000
OK
redis 127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7002
"bar"
redis 127.0.0.1:7000> get hello
-> Redirected to slot [866] located at 127.0.0.1:7000
"world"
```

* 如果我们关闭端口7002对应的redis，则有7002（master节点）对应的7004（slave节点）提供数据。

```sehll
127.0.0.1:7000> get foo
-> Redirected to slot [12182] located at 127.0.0.1:7004
"bar"
```

* 查询分支状态

```shell
$ 127.0.0.1:7004> cluster nodes
21**80 127.0.0.1:7005@17005 slave 92**1b 0 1564994278286 6 connected
50**6f 127.0.0.1:7002@17002 master,fail - 1564994224561 1564994222000 3 connected
23**41 127.0.0.1:7001@17001 master - 0 1564994278085 2 connected 5461-10922
5**f3 127.0.0.1:7003@17003 slave 23**741 0 1564994278000 4 connected
92**1b 127.0.0.1:7000@17000 master - 0 1564994278000 1 connected 0-5460
35**ea 127.0.0.1:7004@17004 myself,master - 0 1564994277000 7 connected 10923-16383
# 此时7004以及编程master节点
```

* 关闭redis节点

```shell
# 可以直接在redis客户端关闭节点
./redis-cli -c -p 7000
127.0.0.1:7000> shutdown
```

或者编写脚本统一关闭所有节点

```shell
# 这里的脚本我放在和启动节点的脚本同一目录（/usr/redis-5.0.4/cluster-test/）
[root@vm cluster-test]# vi stop_all.sh
../src/redis-cli -p 7000 shutdown
../src/redis-cli -p 7001 shutdown
../src/redis-cli -p 7002 shutdown
../src/redis-cli -p 7003 shutdown
../src/redis-cli -p 7004 shutdown
../src/redis-cli -p 7005 shutdown
```

## 二、在docker上部署

* 编辑配置文件`redis.conf`

```shell
port 6379
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

* 建立网段

```shell
docker network create --subnet=172.19.0.0/16 redis-network
```

* 启动redis容器use own redis.conf[docker hub](https://hub.docker.com/_/redis)

```sehll
docker run -v /home/redis/redis.conf:/usr/local/etc/redis/redis.conf -p 5001:6379 --net=redis-network --ip 172.19.0.2 --name r1 -d redis \
redis-server /usr/local/etc/redis/redis.conf
```

* 进入容器

```shell
docker exec -it r1 bash
```

* ~~找到容器中redis执行文件`redis-cli`、`redis-server~~`

```sehll
# 在同一个文件下/usr/local/bin/
find / -name redis-cli
find / -name redis-server
# 并cp到指定目录下
cp redis-server /usr/redis/
```

* 按照前文类似的思路启动其他节点r2、r3、r4、r5、r6.

* 创建集群

```shell
redis-cli --cluster create 172.19.0.2:6379 172.19.0.3:6379 \
172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379 \
--cluster-replicas 1
```

