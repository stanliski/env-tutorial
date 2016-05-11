# MySQL CentOS 6.7 安装

### 安装MySQL Community repository

可以通过以下指令安装`MySQL Community repository`, 命令如下

```
wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
rpm -Uvh mysql-community-release-el6-5.noarch.rpm
```

### 安装 MySQL 5.6

输入以下口令安装 MySQL 5.6

 

yum -y install mysql mysql-server

 

Type in the below to verify the correct packages were installed:

 

rpm -qa | grep mysql