
# Python CentOS 6.7 环境安装

#### Step 1: 安装GCC
首先确认是否安装好了GCC， 假如没有GCC程序，通过以下口令进行安装

```bash
# yum install gcc
```

#### Step 2: 下载 Python 2.7

通过以下口令从[python](https://www.python.org/)官网下载Python 2.7 

```bash
cd /usr/src
wget https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tgz
```

#### Step 3: 解压编译

使用以下口令进行解压编译

```bash
tar xzf Python-2.7.10.tgz
cd Python-2.7.10
./configure
make altinstall
```
> `make altinstall` 使用是为了防止覆盖系统中默认的python环境`/usr/bin/python`.