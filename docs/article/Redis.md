# Redis

> 根据学习慕课网—[一站式学习Redis  从入门到高可用分布式实践](https://coding.imooc.com/class/151.html)，作者是CacheCloud开源项目、《Redis开发与运维》作者。

## 1、Redis简介

> `redis典型应用场景：`缓存系统、计数器、消息队列系统、排行榜、社交网络、实时系统

redis采用单线程，单线程为什么这么块？原因有`1、纯内存。（最主要）`2、非阻塞IO。3、避免多线程使用不合理出现的线程切换和消耗。



## 2、Redis API的理解和使用

### 1、通用命令

keys、dbsize、exists key、del key[key ...]、 expire key seconds、type key

### 2、string（字符串）

计数：incr、decr、incrby、decrby

设置：set、setnx（不存key在则添加）、setxx（存在key即更新）

批量操作：mget、mset（1次mget=1次网络时间+n次命令时间）

### 3、hash（哈希）

所有命令以h开头，大部分命令和string类似

> hash键值结构（可以理解为map的map）：
>
> key	===>> 	(filed------->value)

**应用：**

1、访问量：hincrby user:1:info pageview count

user:1:info --> 用户信息

pageview --> field属性

count --> field对应的值

### 4、list（列表）

> 结构：key	===>> 	element(a-b-c-d)
>
> 特点：有序、可重复、左右对称的操作。
>
> 从左至右索引从0开始、从右至左从-1开始，无论怎样索引，右边大于左边

所有命令以l开头

增：rpush、lpush、linsert（linsert key after|before value newValue）

删：lpop、rpop、lrem（lrem key count  value：count=0，删除所有，count>0从左开始删，count<0从右边开始删）、ltrim（ltrim key start end（包括end）） 

查：lrange（lange key start end（包括end））、lindex、llen

该：lset

**应用：**

lpush + lpop = stack

lpush + rpop = queue

### 5、set（集合）

> 结构： key	===>>	element（集合）
>
> 特点：无序、不可重复、存在集合间的操作

所有命令以s开头

基本：sadd、srem

查找：scard、sismember、srandmember（可以随机挑选指定个数的元素、并且不破坏集合，与spop存在差异）、smembers

集合间：sinter、sdiff、sunion

**应用：**

sadd = tagging

spop、srandmember = random item（如抽奖）

sadd + sinter = social graph （共同关注）

### 6、zset（有序集合）

> 结构： key	===>>	(score ---> element)
>
> 特点：有序、无重复元素（即有顺序的集合）

所有命令以z开头

基本：zadd、zrem、zcard、zincrby、zscore

范围：zrange、zrangbyscore、zcount、zremrangbyrank

集合：zunionstore、zinterstore

**应用：**

排行榜



## 3、Redis Java客户端—Jedis

###  1、连接方案对比

| #      | 优点                                                         | 缺点                                                         |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 直连   | * 简单方便                                                           * 适用于少量长期连接 | * 存在每次新建/关闭而ecp开销                                   *  资源无法控制，存在连接池泄露可能、不安全 |
| 连接池 | * jedis预先生成，降低开销                                         * 连接池的形式报货和控制资源的开销 | * 相对于直连使用更加麻烦，需要合理配置相关参数               |

### 2、连接池参数设置

* maxTotal：资源池最大连接数，默认值为8。
  * 客户端：可根据业务希望的redis并发量（如:50000 QPS）、客户端执行的命令时间（如：平均执行时间为0.1ms = 0.001s）、建议maxTotal > 50000 * 0.001s = 50
  * 服务端：nodes（应用个数）*maxTotal 不能超过redis的最大连接数
* maxIdle：资源池允许的最大空闲连接数，默认值为8。
  * 建议与maxTotal相等
* minIdle：资源池确保最少空闲连接数，默认为0。
* jmxEnabled：是否开启jmx监控，可用于监控，默认为true，建议开启
* blockWhenExhausted：当资源池用尽后，调用者是否要等待。默认值为true，建议开启。
* maxWaitMillis：blockWhenExhausted开启后，该参数有效。当资源池连接用尽后，调用者的最大等待时间（单位为毫秒）。默认值为-1（表示永不超时），不建议使用默认值。
* testOnBorrow：像资源池借用连接时是否做连接有效检测（ping），无效连接会被移除，默认值为false，建议使用默认值。
* testOnReturn：返回连接时。

### 3、代码

```java
JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
jedisPoolConfig.setMaxTotal(10);
jedisPoolConfig.setMaxWaitMillis(1000);

JedisPool jedisPool = new JedisPool(jedisPoolConfig, "192.168.36.111", 6379);

Jedis jedis = null;

try{
    jedis = jedisPool.getResource();
    jedis.ping();
}catch (Exception e){
    //logger.error(e.getMessage(), e);
}finally {
    if(jedis != null){
        jedis.close();
    }
}
```





## 4、Redis 其他功能

* 慢查询
* pipeline
  * 1次pipeline（n条命令时间）= 1次网络时间 + n次命令时间
  * pipeline每次只能作用在一个redis节点上
  * M操作具有原子性、pipeline区别
* 发布订阅
  * 角色：发布者、订阅者、频道
  * API：public、subscribe、unsubscribe
* bitmap：位图
* hyperloglog
  * pfadd key element：向hyperloglog添加元素
  * pfcount key [key ...] ：计算hyperloglog的独立总数
  * pfmerge destkey sourcekey [sourcekey ...]：合并多个hyperloglog
* geo：地理位置



## 5、Redis持久化

持久化（断电不丢数据）：`redis所有数据保存在内存中，对数据的更新将异步的保存在磁盘中。`

 狭义的理解：”持久化”仅仅指把域对象永久保存到数据库中；广义的理解,“持久化”包括和数据库相关的各种操作(持久化就是将有用的数据以某种技术保存起来,将来可以再次取出来应用)。

redis两种持久化方式：`RDB、AOF`。

### 1、RDB方式

RDB主要三种触发方式：sava、bgsave、自动

* RDB时redis内存到硬盘的快照，用于持久化
* sava通常会阻塞redis
* bgsave不会阻塞redis，但是会fork新进程
* sava自动配置满足任意条件就会被执行，不建议使用

```shell
# 关闭默认配置
# save 900 1
# save 300 10
# save 60 10000

stop-writes-on-bgsave-error yes

rdbcompression yes

rdbchecksum yes

# 修改名称
dbfilename dump-${port}.rdb

#指定data目录
dir /opt/soft/redis/data
```

**存在问题：**

耗时、好性能：data dump to disk

不可控、丢失数据：dump没完成时，此时写入新数据出现宕机，数据丢失。

### 2、AOF方式

AOF主要三种触发方式：always、`everysec`、no

#### AOF重写

重写过程中，fock子进程，重新写入的数据也会并入到新的AOF文件中。

* AOF重写触发：
  * bgrewriteaof
  * 满足条件自动触发
* **AOF重写作用**：减少硬盘占用量、加速恢复速度

```shell
# 修改
appendonly yes

# 修改
appendfilename "appendonly-6379.aof"

# If unsure, use "everysec".
# appendfsync always
appendfsync everysec
# appendfsync no

# 修改:为了提高性能，选择丢失部分数据
no-appendfsync-on-rewrite yes

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-load-truncated yes

aof-use-rdb-preamble yes
```

### 3、RDB与AOF

| #          | RDB          | AOF          |
| ---------- | ------------ | ------------ |
| 启动优先级 | 低           | 高           |
| 体积       | 小（二进制） | 大           |
| 恢复速度   | 块           | 慢           |
| 数据安全性 | 丢数据       | 根据策略决定 |
| 轻重       | 重           | 轻           |

**决策：RDB"关“、AOF”开“**



## 6、Redis主从复制

```shell
slaveof ip port
slave-read-only yes
```



## 7、Redis Sentinel

用于解决redis主从复制（高可用）过程中redis-master节点宕机出现的问题，用于监控master-slave集群，实现master节点故障自动转移。

> 哨兵模式：Sentinel其实就是Client和Redis之间的桥梁，所有的客户端都通过Sentinel程序获取Redis的Master服务。。**首先Sentinel是集群部署的，Client可以链接任何一个Sentinel服务所获的结果都是一致的。其次，所有的Sentinel服务都会对Redis的主从服务进行监控，当监控到Master服务无响应的时候，Sentinel内部进行仲裁，从所有的 Slave选举出一个做为新的Master。并且把其他的slave作为新的Master的Slave。**

### 1、Redis Sentinel failover

* sentinel monitor <master-name> <ip> <redis-port> <quorum>指定监控的主节点
* sentinel down-after-milliseconds mymaster 30000 某个sentinel对redis节点失败的“偏架”，即"主观下线"。
* “偏见”数量大于 <quorum>时，所有sentinel达成共识，即“客观下线”
* sentinel集群中，选举出一个领导者节点
* sentinel领导者节点，实现故障转移，也就是选举出一个新的master节点。（1、优先级最好的slave节点。2、寻找偏移量最大——复制最完整的节点。3、选择runid最小的节点——最好启动的节点）

### 2、jedis客户端—JedisSentinelPool

```java
String masterName = "mymaster";
Set<String> sentinels = new HashSet<String>();
sentinels.add("192.168.36.111:26379")
    sentinels.add("192.168.36.111:26380")
    sentinels.add("192.168.36.111:26381")

    JedisSentinelPool jedisSentinelPool = new JedisSentinelPool(masterName, sentinels);
```



## 8、Redis Cluster

### 1、只是储备——数据分布

#### 1、数据分区

顺序分区、哈希分区：数据分散度高，主要运用与缓存。

#### 2、哈希分布

* 节点取余分区
  * 客户端分片：哈希 + 取余
  * 数据迁移量大
* 一致性哈希分区
  * 客户端分片：哈希 + 顺时针（对取余进行优化）
  * 相对于节点取余，数据前移量降低
* 虚拟槽分区
  * 服务端管理节点、槽、数据：例如 Redis Cluster 
  * 良好的哈希函数：例如CRC16
  * 预设虚拟槽：每个槽映射一个数据子集（16384个槽）

### 2、集群

> [redis cluster官方文档](https://redis.io/topics/cluster-tutorial)

Redis Cluster架构：节点、meet、指派槽、复制

Redis Cluster特性：复制高可用、分片

Redis Cluster部署：参考[redis集群部署](article/redis集群部署.md)



**smart客户端——JedisCluster**

```java
Set<HostAndPort> nodeList = new HashSet<HostAndPort>();
nodeList.add(new HostAndPort(host, poat));
...
JedisCluster jedisCluster = new JedisCluster(nodeList, timeout, poolConfig);
jedisCluster.command...
```



## 9、缓存设计

* 使用场景
  * 降低后端负载
  * 加速请求响应
  * 大量写合并为批量写
* 缓存更新问题
* 缓存粒度问题
  * 通用性：全量属性最好
  * 占用空间：部分属性最好
  * 代码维护：表面上全量属性最好
* 缓存穿透问题
  * 返回空键值
  * 在缓存层之上添加bloom过滤器
* 缓存雪崩问题
  * 保证高可用：1、机房；2、redis cluster、redis sentinel、 vip



## 10、Redis云平台CacheCloud（跳过）



## 11、Redis布隆过滤器

> 现有50亿电话号码，如何快速判断10w个电话号码是否在其中？

利用bloom过滤器，用较小的过程，实现在海量数据中准确快速的查找指定的数据。



## 12、Redis开发规范

### 1、key名设计

* **可读性和可管理性：**以业务表为前缀，用冒号分割，例如：ugc:video:1。
* **简洁性：**保证语义的前提下，控制key的长度降低内存消耗，如：messages --> m
* **不要包含特殊字符：**

### 2、value名设计

* **解决bigkey**
  * string类型控制在10kb以内
  * hash、list、set、zset元素个数不要超过5000个
  * 反例：一个包含几百万个元素的list、hash等，一个巨大的json字符串。
* **选择合适的数据结构**
* **过期设计**

### 3、命令使用技巧

1. O(N)以上的命令关注N的数量
2. 禁用命令（如：keys、flushdb、flushall等，通过redisrename机制禁掉命令）
3. 合理使用select：redis的多数据库较弱
4. redis事务功能较弱，不建议使用
5. redis集群版本在使用lua上有特殊要求
6. 必要情况下使用monitor命令，但不要长时间使用

### 4、java客户端优化

#### 1、避免多个应用使用一个redis实例

不相干的业务拆分，公共数据做服务化

#### 2、使用连接池

#### 3、连接池参数优化

基本参数1：见[2、连接池参数设置](###2、连接池参数设置)

基本参数2：testWhileIdle默认为false，建议使用true开启；其他资源空闲相关配置建议使用JedisPoolConfig中的配置。



## 13、内存管理（跳过）