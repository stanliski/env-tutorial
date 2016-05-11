# MySQL CentOS 6.7 安装

### 安装MySQL Community repository

可以通过以下指令安装`MySQL Community repository`, 命令如下

```
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
rpm -Uvh mysql-community-release-el6-5.noarch.rpm
```

### 安装 MySQL 5.6

输入以下口令安装 MySQL 5.6

```
yum -y install mysql mysql-server
```
输入以下口令判断是否正确安装： 

```
rpm -qa | grep mysql
```

可能会产生如下输出

```
mysql-community-release-el6-5.noarch
mysql-community-common-5.6.27-2.el6.x86_64
mysql-community-client-5.6.27-2.el6.x86_64
mysql-community-server-5.6.27-2.el6.x86_64
mysql-community-libs-5.6.27-2.el6.x86_64
mysql-community-libs-compat-5.6.27-2.el6.x86_64
```

通过以下指令启动`MySQL 5.6`

```
chkconfig mysqld on
service mysqld start
```
