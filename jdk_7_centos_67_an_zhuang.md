# JDK 7 CentOS 6.7 安装


#### 第一步：卸载删除OpenJDK

卸载删除OpenJDK，首先需要知道到底要删除哪些东西：

```bash
[Randy@localhost ~]$ rpm -qa|grep openjdk -i 
```