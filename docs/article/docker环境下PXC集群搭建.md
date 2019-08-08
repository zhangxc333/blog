# docker环境下pxc集群搭建

## 常用MySQL集群介绍

> ==PXC方案==：==速度慢，强一致性，适合保存高价值数==据，如：订单、用户、财务等。
>
> Replication方案：速度快，弱一致性，适合保存低价值数据，如：新闻、日志、帖子等。

## PXC方案的强一致性

* ==同步复制，事务==在所有集群点中要么同时提交、要么不提交
* replication采用异步复制，无法保证数据的一致性。

## PXC集群配置

根据percona官方网站 [Running Percona XtraDB Cluster in a Docker Container](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/install/docker.html)

> Note
>
> Make sure that you are using the latest version of Docker. The ones provided via `apt` and `yum` may be outdated and cause errors.

1. Create a Docker network:

```shell
# docker内置的默认网段（bridge）是`172.17.0.0/16`，以后开启的网段18.19...依此类推。我们也可以通过==--subnet==来指定特定的网段。
docker network create pxc-network
```

2. Bootstrap the cluster (create the first node):

```shell
docker run -d \
  -e MYSQL_ROOT_PASSWORD=admin \
  -e CLUSTER_NAME=cluster1 \
  -p 3306:3306 \ # 对外开发端口，这里官方文档没有提及
  # -v v1:/var/lib/mysql 映射数据卷
  --name=node1 \
  --net=pxc-network \
  percona/percona-xtradb-cluster:5.7
```

3. Join the second node:

```shell
docker run -d \
  -e MYSQL_ROOT_PASSWORD=root \
  -e CLUSTER_NAME=cluster1 \
  -e CLUSTER_JOIN=node1 \
  -p 3307:3306 \ # 对外开发端口，这里官方文档没有提及
  # -v v1:/var/lib/mysql 映射数据卷
  --name=node2 \
  --net=pxc-network \
  percona/percona-xtradb-cluster:5.7
```

4. Join the third node:

```shell
docker run -d \
  -e MYSQL_ROOT_PASSWORD=root \
  -e CLUSTER_NAME=cluster1 \
  -e CLUSTER_JOIN=node1 \
  -p 3308:3306 \ # 对外开发端口，这里官方文档没有提及
  # -v v1:/var/lib/mysql 映射数据卷
  --name=node3 \
  --net=pxc-network \
  percona/percona-xtradb-cluster:5.7
```

To ensure that the cluster is running:

1. Access the MySQL client. For example, on the first node:

```shell
sudo docker exec -it node1 /usr/bin/mysql -uroot -padmin
```

2. View the wsrep status variables:

```shell
mysql@node1> show status like 'wsrep%';
```