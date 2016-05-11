# MySQL Cluster CentOS 6.7 安装

### MySQL Cluster 架构介绍

MySQL Cluster 是MySQL 适合于分布式计算环境的高实用、可拓展、高性能、高冗余版本，其研发设计的初衷就是要满足许多行业里的最严酷应用要求，这些应用中经常要求数据库运行的可靠性要达到`99.999%`。MySQL Cluster允许在无共享的系统中部署“内存中”数据库集群，通过无共享体系结构，系统能够使用廉价的硬件，而且对软硬件无特殊要求。此外，由于每个组件有自己的内存和磁盘，不存在单点故障。实际上，MySQL集群是把一个叫做NDB的内存集群存储引擎集成与标准的MySQL服务器集
成。它包含一组计算机，每个都跑一个或者多个进程，这可能包括一个MySQL服务器，一个数据节点，一个管理服务器和一个专有的一个数据访问程序。MySQL Cluster能够使用多种故障切换和负载平衡选项配置NDB存储引擎，但在Cluster 级别上的存储引擎上做这个最简单。以下为MySQL集群结构关系图

MySQL从结构看，由3类节点(计算机或进程)组成，分别是：
- `管理节点`:用于给整个集群其他节点提供配置、管理、仲裁等功能。理论上通过一台
服务器提供服务就可以了。
- `数据节点`: MySQL Cluster的核心，存储数据、日志，提供数据的各种管理服务。2个
以上 时就能实现集群的高可用保证，DB节点增加时，集群的处理速度会变慢。
- `SQL节点(API)`: 用于访问MySQL Cluster数据，提供对外应用服务。增加 API 节点会
提高整个集群的并发访问速度和整体的吞吐量，该节点 可以部署在Web应用服务器
上，也可以部署在专用的服务器上，也开以和DB部署在同一台服务器上。

### NDB引擎

MySQL Cluster 使用了一个专用的基于内存的存储引擎——NDB引擎，这样做的好处是速度快， 没有磁盘I/O的瓶颈，但是由于是基于内存的，所以数据库的规模受系统总内存的限制， 如果运行NDB的MySQL服务器一定要内存够大，比如4G, 8G, 甚至16G。NDB引擎是分布式的，它可以配置在多台服务器上来实现数据的可靠性和扩展性，理论上 通过配置2台NDB的存储节点就能实现整个数据库集群的冗余性和解决单点故障问题。

#### 缺陷
- 基于内存，数据库的规模受集群总内存的大小限制
- 基于内存，断电后数据可能会有数据丢失，这点还需要通过测试验证。
- 多个节点通过网络实现通讯和数据同步、查询等操作，因此整体性受网络速度影响，
因此速度也比较慢

#### 优点
- 多个节点之间可以分布在不同的地理位置，因此也是一个实现分布式数据库的方案。
- 扩展性很好，增加节点即可实现数据库集群的扩展。
- 冗余性很好，多个节点上都有完整的数据库数据，因此任何一个节点宕机都不会造成服务中断。
- 实现高可用性的成本比较低，不象传统的高可用方案一样需要共享的存储设备和专用的软件才能实现，NDB 只要有足够的内存就能实现。

#### 安装要求
- 安装环境 ：CentOS-6.3
- 安装方式 ：源码编译安装
- 软件名称 ：mysql-cluster-gpl-7.2.6-linux2.6-x86_64.tar.gz
- 下载地址 ：http://mysql.mirror.kangaroot.net/Downloads/
- 软件安装位置 ：/usr/local/mysql
- 数据存放位置 ：/var/mysql/data
- 数据存放位置 ：/var/mysql/logs


### 虚拟机角色分配

首先设计集群的安装分配方式，我共需要5台服务器，服务器分配如下：
- 管理节点： 192.168.2.250
- sql节点1： 192.168.2.251
- sql节点2： 192.168.2.252
- 数据节点1： 192.168.2.253
- 数据节点2： 192.168.2.254

### 检查安装的mysql

检查系统中已经安装过的mysql信息，操作如下：

```
[root@localhost /]# rpm -qa | grep mysql
[root@localhost /]# service mysql status
```

如果安装过其他版本的mysql，请卸载，操作如下：

```
/etc/init.d/mysqld stop //关闭目前的mysql服务
ps -ef | grep mysql //检验mysql是否已经关闭
rpm -e --allmatches --nodeps mysql mysql-server
rm -rf /var/lib/mysql // 删除mysql的安装目录
```

### 管理节点
#### 管理节点安装
安装管理节点（192.168.15.231）

```
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd mysql -g mysql
[root@localhost /]# cd /usr/local
[root@localhost local]# tar -zxv -f mysql-cluster-gpl-7.2.6-linux
2.6-x86_64.tar.gz
[root@localhost local]# mv mysql-cluster-gpl-7.2.6-linux2.6-x86_64
mysql
[root@localhost local]# chown -R mysql:mysql mysql
[root@localhost local]# cd mysql
[root@localhost mysql]# scripts/mysql_install_db --user=mysql
```

#### 管理节点配置

```
[root@localhost ~]# mkdir /var/lib/mysql-cluster
[root@localhost ~]# cd /var/lib/mysql-cluster
[root@localhost mysql-cluster]# vim /var/lib/mysql-cluster/config.i
ni
```

在`config.ini`中添加以下内容:

```
[NDBD DEFAULT]
NoOfReplicas=1
[TCP DEFAULT]
portnumber=3306
[NDB_MGMD]
HostName=192.168.2.250
DataDir=/var/mysql/data
[NDBD]
HostName=192.168.2.251
DataDir=/var/mysql/data
[NDBD]
HostName=192.168.2.252
DataDir=/var/mysql/data
[MYSQLD]
HostName=192.168.2.253
[MYSQLD]
HostName=192.168.2.254
```

#### 管理节点启动

```
[root@localhost ~]# /usr/local/mysql/bin/ndb_mgmd -f /var/lib/mysq
l-cluster/config.ini
[root@localhost ~]# mkdir /var/mysql/logs
[root@localhost ~]# netstat -lntpu
```
看到`tcp 0 0 0.0.0.0:1186`开放说明启动正常
开启管理节点服务器的`1186`端口

#### 管理节点检验

执行以下操作：

```
[root@localhost /]# ndb_mgm // 管理节点
-- NDB Cluster -- Management Client --
ndb_mgm> show
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)] 2 node(s)
id=2 (not connected, accepting connect from 192.168.2.251)
id=3 (not connected, accepting connect from 192.168.2.252)
[ndb_mgmd(MGM)] 1 node(s)
id=1 @192.168.2.250 (mysql-5.5.22 ndb-7.2.6)
[mysqld(API)] 2 node(s)
id=4 (not connected, accepting connect from 192.168.2.253)
id=5 (not connected, accepting connect from 192.168.2.254)
```

#### 管理节点关闭

```
[root@localhost /]# /usr/local/mysql/bin/ndb_mgm -e shutdown
Connected to Management Server at: 192.168.2.250:1186
3 NDB Cluster node(s) have shutdown.
Disconnecting to allow management server to shutdown.
```

### 数据节点

#### 数据节点安装

- 数据节点1： 192.168.2.251
- 数据节点2： 192.168.2.252

```
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd mysql -g mysql
[root@localhost /]# cd /usr/local
[root@localhost local]# tar -zxv -f mysql-cluster-gpl-7.2.6-linux
2.6-x86_64.tar.gz
[root@localhost local]# mv mysql-cluster-gpl-7.2.6-linux2.6-x86_64
mysql
[root@localhost local]# chown -R mysql:mysql mysql
[root@localhost local]# cd mysql
[root@localhost mysql]# scripts/mysql_install_db --user=mysql
[root@localhost mysql]# cp support-files/my-medium.cnf /etc/my.cnf
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/m
ysqld
```

#### 数据节点配置

对数据节点进行配置，执行以下操作： 

```
[root@localhost mysql]# mkdir /var/mysql/data
[root@localhost mysql]# mkdir /var/mysql/logs
[root@localhost mysql]# vi /etc/my.cnf
```
向文件追加以下内容：

```
[MYSQLD]
ndbcluster
ndb-connectstring=192.168.2.250
[MYSQL_CLUSTER]
ndb-connectstring=192.168.2.250
[NDB_MGM]
connect-string=192.168.2.250
```

#### 数据节点启动

启动此处时，管理节点服务器防火墙必须开启1186,3306端口。
> 注意：只是在第一次启动或在备份／恢复或配置变化后重启ndbd时，才加–initial参数！
第一次启动如下：

```
[root@localhost mysql]# /usr/local/mysql/bin/ndbd --initial
2013-01-30 13:43:53 [ndbd] INFO -- Angel connected to '192.16
8.2.250:1186'
2013-01-30 13:43:53 [ndbd] INFO -- Angel allocate
```

正常启动方式： 

```
[root@localhost mysql]# /usr/local/mysql/bin/ndbd
```

#### 数据节点关闭

```
[root@localhost /]# /etc/rc.d/init.d/mysqld stop
or
[root@localhost mysql]# /etc/init.d/mysql stop
Shutting down MySQL.. SUCCESS!
/usr/local/mysql/bin/mysqladmin -uroot shutdown
```

### SQL节点安装

SQL节点和存储节点(NDB节点)安装相同，都执行以下操作；
- sql节点1： 192.168.2.253
- sql节点2： 192.168.2.254

```
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd mysql -g mysql
[root@localhost /]# cd /usr/local
[root@localhost local]# tar -zxv -f mysql-cluster-gpl-7.2.6-linux
2.6-x86_64.tar.gz
[root@localhost local]# mv mysql-cluster-gpl-7.2.6-linux2.6-x86_64
mysql
[root@localhost local]# chown -R mysql:mysql mysql
[root@localhost local]# cd mysql
[root@localhost mysql]# scripts/mysql_install_db --user=mysql
[root@localhost mysql]# cp support-files/my-medium.cnf /etc/my.cnf
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/m
ysqld
```

#### SQL节点配置

执行以下操作：

```
[root@localhost mysql]# mkdir /var/mysql/data //创建存储数据的文件
夹
[root@localhost mysql]# mkdir /var/mysql/logs //创建存储日志的文件
夹
[root@localhost mysql]# vi /etc/my.cnf //修改配置文件
```

追加以下内容：

```
[MYSQLD]
ndbcluster
ndb-connectstring=192.168.2.250
[MYSQL_CLUSTER]
ndb-connectstring=192.168.2.250
[NDB_MGM]
connect-string=192.168.2.250
```

#### SQL节点启动

执行以下操作：

```
[root@localhost mysql]# service mysqld start
Starting MySQL.. SUCCESS!
```

#### SQL节点关闭

最直接的方式：

```
[root@localhost mysql]# /usr/local/mysql/bin/mysqladmin -uroot shut
down
```

或者

```
[root@localhost /]# /etc/rc.d/init.d/mysqld stop
[root@localhost mysql]# /etc/init.d/mysql stop
Shutting down MySQL.. SUCCESS!
```

#### 在每个SQL节点修改数据库密码

初始化root密码

```
update user set password=PASSWORD(‘NewPassword’) where Use
r='root';
```

数据库输入密码报错

```
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (usin
g password: NO)
```

```
set password for 'root'@'localhost' = password('NewPassword');
flush privileges;
```

### 功能测试

在管理节点（192.168.2.250）上查看服务状态

```
[root@localhost ~]# /usr/local/mysql/bin/ndb_mgm
ndb_mgm> show
Cluster Configuration
```

output:
```
[ndbd(NDB)] 2 node(s)
id=2 @192.168.15.234 (mysql-5.5.22 ndb-7.2.6, Nodegroup: 0, Mas
ter)
id=3 @192.168.15.235 (mysql-5.5.22 ndb-7.2.6, Nodegroup: 1)
[ndb_mgmd(MGM)] 1 node(s)
id=1 @192.168.15.231 (mysql-5.5.22 ndb-7.2.6)
[mysqld(API)] 2 node(s)
id=4 @192.168.15.232 (mysql-5.5.22 ndb-7.2.6)
id=5 @192.168.15.233 (mysql-5.5.22 ndb-7.2.6)
```

可以看到这里的数据节点、管理节点、sql节点都是正常的。

#### 数据同步性测试

在一个数据节点上进行相关数据库的创建，然后到另外一个数据节点上看看数据是否同步。
- 第1步：
SQL节点1（192.168.2.253）上增加数

```
[root@localhost mysql]# /etc/rc.d/init.d/mysqld status
//检验mysql是否运行
[root@localhost mysql]# /etc/rc.d/init.d/mysqld start
//启动mysql
[root@localhost mysql]# /usr/local/mysql/bin/mysql -u root -p
Enter password:
mysql> show databases;
mysql> create database testdb2;
mysql> use testdb2;
mysql> CREATE TABLE td_test2 (i INT) ENGINE=NDB;
//这里必须指定数据库表的引擎为NDBCLUSTER，与配置文件中的名称相同
mysql> INSERT INTO td_test2() VALUES (1);
mysql> INSERT INTO td_test2() VALUES (152);
mysql> SELECT * FROM td_test2;
```

- 第2步：
进入到SQL节点2（192.168.2.254）上查看数据

```
mysql> use testdb2;
Database changed
mysql> SELECT * FROM td_test2;
+------+
| i |
+------+
| 126 |
| 1 |
+------+
2 rows in set (0.01 sec)
```

查看表的引擎是不是NDB：

```
show create table td_test2;
```

- 第3步： 反向测试，SQL节点2（192.168.2.254）上增加数据：

```
mysql> create database bb;
mysql> use bb;
mysql> CREATE TABLE td_test3 (i INT) ENGINE=NDB;
mysql> INSERT INTO td_test3 () VALUES (98);
mysql> SELECT * FROM td_test3;
```

SQL节点1（192.168.2.253）上查看数据：

```
mysql> use bb;
Database changed
mysql> SELECT * FROM td_test3;
+------+
| i |
+------+
| 98 |
+------+
1 row in set (0.00 sec)
```

####关闭集群
先关闭管理节点，然后关闭SQL节点和数据节点。
####集群启动操作顺序
要再次启动集群，按照以下顺序执行：
管理节点 -> 数据节点 –> SQL节点 
> 注意：此次启动数据节点时不要加”–initial”参数。

### 安装及测试中的错误

启动中的错误
错误提示：

```
Can't connect to local MySQL server through socket '/tmp/mysql.soc
k' (2)
```

解决办法1（端口占用）

```
netstat -anp |grep 3306
kill -9 进程号
```

解决办法2（权限问题）

```bash
[root@localhost mysql]# chown -R mysql:mysql /var/mysql
//修改自定义文件夹的访问权限
```