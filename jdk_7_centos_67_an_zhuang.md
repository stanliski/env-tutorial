# JDK 7 CentOS 6.7 安装


#### 第一步：卸载删除OpenJDK

卸载删除OpenJDK，首先需要知道到底要删除哪些东西：

```bash
$ rpm -qa|grep openjdk -i 
```

现在将之全部删除：

```bash 
$ sudo yum remove java-1.6.0-openjdk-devel-1.6.0.0-6.1.13.4.el7_0.x86_64 java-1.7.0-openjdk-devel-1.7.0.65-2.5.1.2.el7_0.x86_64 java-1.7.0-openjdk-headless-1.7.0.65-2.5.1.2.el7_0.x86_64 java-1.7.0-openjdk-1.7.0.65-2.5.1.2.el7_0.x86_64 java-1.6.0-openjdk-1.6.0.0-6.1.13.4.el7_0.x86_64
```

#### 第二步：安装JDK

#### 1、解压
首先解压下载得来的JDK：（JDK的tar.gz压缩包放在了~/dev目录下）

```
$ sudo mkdir /usr/lib/jdk #如若没有/usr/lib/jdk路径，则执行此句予以创建jdk文件夹
$ sudo tar -zxvf jdk-8u11-linux-i586.tar.gz -C /usr/lib/jdk #注意：-C, --directory=DIR        改变至目录 DIR
$  ls /usr/lib/jdk
jdk1.8.0_11
$ ls /usr/lib/jdk/jdk1.8.0_11/
bin        javafx-src.zip  man          THIRDPARTYLICENSEREADME-JAVAFX.txt
COPYRIGHT  jre             README.html  THIRDPARTYLICENSEREADME.txt
db         lib             release
include    LICENSE         src.zip
[Randy@localhost ~]$

```

#### 2、配置环境变量

```bash
$ sudo vim /etc/profile
#JAVA Environment
export JAVA_HOME=/usr/lib/jdk
export JRE_HOME=/usr/lib/jdk/jre
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JRE_HOME/lib
```

#### 3、修改系统默认的JDK

```bash
$ sudo update-alternatives --install /usr/bin/java java /usr/lib/jdk/bin/java 300  #使系统默认的java命令是/usr/lib/jdk/bin中的java命令
$ sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jdk/bin/javac 300  #使系统默认的javac命令是/usr/lib/jdk/bin中的javac命令
$ sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jdk/bin/jar 300 #使系统默认的jar命令是/usr/lib/jdk/bin中的jar命令 
$ sudo update-alternatives --config java   #配置默认java命令

共有 1 个提供“java”的程序。
  选项    命令
-----------------------------------------------
*+ 1          /usr/lib/jdk/bin/java
          
按 Enter 保留当前选项[+]，或者键入选项编号：1

[Randy@localhost ~]$ sudo update-alternatives --config javac   #配置默认java命令
共有 1 个提供“java”的程序。

  选项    命令
-----------------------------------------------
*+ 1          /usr/lib/jdk/bin/javac
          
按 Enter 保留当前选项[+]，或者键入选项编号：1

```