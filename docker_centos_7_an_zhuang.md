# Docker CentOS 7 安装

#### 1.在CentOS7上安装Docker

在默认的CentOS-Extra库中已经包含了Docker安装包。所以安装Docker，仅仅需要简单的
Base Shell指令：

```bash
yum install docker
```

#### 2.安装完毕后，启动Docker服务

一旦安装完成，就可以启动相应的Docker服务

```bash
service docker start
```

设置开机自启动 

```bash
chkconfig docker on
```
