# pip Centos 6.7 环境安装


#### 安装PIP

在安装pip之前，首先得确认电脑已经包含了python 2.7环境(Centos7自带python 2.7)。
并且还要安装EPEL包

#### RHEL 7.x 和 CentOS 7.x (x86_64) 环境EPEL安装

可以直接通过yum指令进行安装

```bash
$ yum install epel-release
```

#### RHEL 6.x 和 CentOS 6.x (x86_64)环境EPEL安装

对于7以下的版本，可以通过rpm进行安装

```bash
$ rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-relea
se-6-8.noarch.rpm
```

#### 通过yum安装pip

安装指令如下：

```bash
$ yum install -y python-pip
```