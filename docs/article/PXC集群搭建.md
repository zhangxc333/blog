# PXC集群搭建

> [Percona XtraDB Cluster 5.7 Documentation](https://www.percona.com/doc/percona-xtradb-cluster/5.7/index.html)

## 一、安装

> 根据官方文档[Installing *Percona Server* ](https://www.percona.com/doc/percona-server/5.7/installation/yum_repo.html)、[Installing Percona XtraDB Cluster ](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/install/yum.html#yum)on Red Hat Enterprise Linux and CentOS。

1. Install the Percona repository

```shell
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

To install *Percona Server* with SELinux policies, you also need the **Percona-Server-selinux-\*.noarch.rpm** package:

```shell
yum install http://repo.percona.com/centos/7/RPMS/x86_64/Percona-Server-selinux-56-5.6.42-rel84.2.el7.noarch.rpm
```

2. Testing the repository

Make sure packages are now available from the repository, by executing the following command:

```
yum list | grep percona
```

3. Install the packages

```shell
# You can now install Percona Server by running:
yum install Percona-Server-server-57
```



```shell
# sudo yum install Percona-XtraDB-Cluster-57
sudo yum install Percona-XtraDB-Cluster-57
```

## 二、启动pxc节点

> Note
>
> Make sure that the following ports are not blocked by firewall or used by other software. Percona XtraDB Cluster requires them for communication.
>
> - 3306
> - 4444
> - 4567
> - 4568
>
> Note
>
> The [SELinux](https://selinuxproject.org/) security module can constrain access to data for Percona XtraDB Cluster. The best solution is to change the mode from `enforcing` to `permissive` by running the following command:
>
> ```
> setenforce 0
> ```
>
> This only changes the mode at runtime. To run SELinux in permissive mode after a reboot, set `SELINUX=permissive` in the `/etc/selinux/config` configuration file.

1. Start the Percona XtraDB Cluster server:

```shell
sudo service mysql start
```

2. Copy the automatically generated temporary password for the superuser account:

```shell
sudo grep 'temporary password' /var/log/mysqld.log
```

3. Use this password to log in as `root`:

```shell
mysql -u root -p
```

4.  Change the password for the superuser account and log out. For example:

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootPass';
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```

5. Stop the `mysql` service:

```shell
sudo service mysql stop
```

## 三、Configuring Nodes for Write-Set Replication

> Note
>
> Make sure that the Percona XtraDB Cluster server is not running.

按照启动pxc节点的方式，启动三个节点

| Node   | Host | IP            |
| :----- | :--- | :------------ |
| Node 1 | pxc1 | 192.168.8.127 |
| Node 2 | pxc2 | 192.168.8.128 |
| Node 3 | pxc3 | 192.168.8.129 |

If you are running Red Hat or CentOS, add the following configuration variables to `/etc/my.cnf` on the first node:

```shell
# 实际过程进入 vi /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 进行相应的修改即可

[mysqld]
# Path to Galera library
wsrep_provider=/usr/lib64/galera3/libgalera_smm.so

# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node

# 修改未启动节点的ip
wsrep_cluster_address=gcomm://192.168.8.128,192.168.8.127,192.168.8.129

# In order for Galera to work correctly binlog format should be ROW
binlog_format=ROW

# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB

# Slave thread to use
wsrep_slave_threads= 8

wsrep_log_conflicts

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# Node IP address
# 该节点的ip
wsrep_node_address=192.168.8.127

# Cluster name
wsrep_cluster_name=pxc-cluster

#If wsrep_node_name is not specified,  then system hostname will be used
# 修改未对应的名称
wsrep_node_name=pxc-cluster-node-2

#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING

# SST method
wsrep_sst_method=xtrabackup-v2

#Authentication for SST method
# 见下面介绍
wsrep_sst_auth="sstuser:admin"

```

`wsrep_sst_auth`

> Specify authentication credentials for [SST](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/glossary.html#term-sst) as `<sst_user>:<sst_pass>`. You must create this user when [Bootstrapping the First Node](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/bootstrap.html#bootstrap) and provide necessary privileges for it:

```
mysql> CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 'passw0rd';
mysql> GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* TO
  'sstuser'@'localhost';
mysql> FLUSH PRIVILEGES;
```

## 四、搭建集群

* bootstraping the first node 

```shell
[root@pxc1 ~]# systemctl start mysql@bootstrap.service
```

To make sure that the cluster has been initialized, run the following:

```shell
mysql@pxc1> show status like 'wsrep%';
```

Before [adding other nodes](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/add-node.html#add-node) to your new cluster, create a user for [SST](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/glossary.html#term-sst) and provide necessary privileges for it. The credentials must match those specified when [Configuring Nodes for Write-Set Replication](https://www.percona.com/doc/percona-xtradb-cluster/LATEST/configure.html#configure).

* starting the second node

```shell
systemctl start mysql
```

* starting the third node

```shell
systemctl start mysql
```

## 五、节点关闭

* 在安全退出的情况下，最后推出的节点，下次启动时为主节点。
* 如果最后关闭的PXC节点不是安全退出的，那么要先修改`/var/lib/mysql/grastate.dat` 文件，把其中的`safe_to_bootstrap`属性值设置为1，该节点启动就是主节点。