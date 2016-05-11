# JDK 7 CentOS 6.7 安装


#### 第一步：卸载删除OpenJDK

卸载删除OpenJDK，首先需要知道到底要删除哪些东西：

```bash
[Randy@localhost ~]$ rpm -qa|grep openjdk -i 
```

现在将之全部删除：

```bash 
[Randy@localhost ~]$ sudo yum remove java-1.6.0-openjdk-devel-1.6.0.0-6.1.13.4.el7_0.x86_64 java-1.7.0-openjdk-devel-1.7.0.65-2.5.1.2.el7_0.x86_64 java-1.7.0-openjdk-headless-1.7.0.65-2.5.1.2.el7_0.x86_64 java-1.7.0-openjdk-1.7.0.65-2.5.1.2.el7_0.x86_64 java-1.6.0-openjdk-1.6.0.0-6.1.13.4.el7_0.x86_64
```

#### 第二步：安装JDK

#### 1、解压
首先解压下载得来的JDK：（JDK的tar.gz压缩包放在了~/dev目录下）
