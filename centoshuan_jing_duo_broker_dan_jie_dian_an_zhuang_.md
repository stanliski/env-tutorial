# CentOS环境多Broker单节点安装 Apache Kafka 0.8教程


### 搭建Kafka环境描述

在下面的教程中，我将详细介绍如何在单台虚拟机上安装一个多`Broker`的`Apache Kafka`集群。我们也将通过producer客户端发送消息给kafka集群，并通过consumer客户端接收消息:

以下是我们需要做的具体工作：
- 使用单台环境部署Kafka
- 在单台虚拟机上运行`Zookeeper`进行任务调度
- 在单台虚拟机上运行三个`Kafka brokers`
- 创建一个叫做`zerg.hydra`的主题，通过客户端进行发送和接收。主题将使用
3个partition，每个`partition`中包含2个replicas。

### 搭建环境示意图

下图展示了我们需要搭建的多Broker示意图：

![](http://www.michael-noll.com/blog/uploads/kafka-cluster-overview.png)

### 环境安装
#### 下载Zookeeper与kafka
##### 启动Zookeeper

使用Kafka时需要使用Zookeeper进行负载均衡与资源调度。可以通过以下指令启动
Zookeeper：

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```
Zookeeper默认会在 `*:2181/tcp`.端口进行监听

##### 配置和启动Kafka brokers

在这个教程中，我们将会创建3个broker，默认的配置文件是` config/server.properties`

```
cp config/server.properties config/server01.properties
```

编辑 `server01.properties`,修改配置文件中的如下字段：

```
broker.id=1
port=9092
log.dir=/tmp/kafka-logs-1
```

配置第二个Broker

```
cp config/server.properties config/server02.properties
```
编辑 `server02.properties`,修改配置文件中的如下字段

```bash
broker.id=2
port=9093
log.dir=/tmp/ka
```

配置第三个Broker

```bash
cp config/server.properties config/server03.properties
```

编辑 `server03.properties`,修改配置文件中的如下字段：

```bash
broker.id=3
port=9094
log.dir=/tmp/kafka-logs-3
```

配置完成后，依次启动broker

```bash
env JMX_PORT=9999 bin/kafka-server-start.sh config/server01.proper
ties
env JMX_PORT=10000 bin/kafka-server-start.sh config/server02.prope
rties
env JMX_PORT=10001 bin/kafka-server-start.sh config/server03.prope
rties
```

运行后的监听端口和网络协议如下：

```bash
Broker 1 Broker 2 Broker 3
----------------------------------------------
Kafka *:9092/tcp *:9093/tcp *:9094/tcp
JMX *:9999/tcp *:10000/tcp *:10001/tcp
```

### 环境测试

#### 创建一个Kafka Topic

在以下命令中创建了一个 “zerg.hydra”的新主题。这个主题使用了3个partitions，并且每个
partition包含两个replication。

```bash
bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic zer
g.hydra --partitions 3 --replication-factor 2
```
运行成功后，kafka集群会为topic创建三个逻辑partition。kafka集群也会为每个partition创
建两个replicas(拷贝)。

#### 查看已创建的Kafka Topic

运行以下Command，可以获取当前创建的topic

```
bin/kafka-topics.sh --zookeeper localhost:2181 --list
<snipp>
zerg.hydra
</snipp>
```

运行以下Command，可以查看具体的某个topic信息

```
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic z
erg.hydra
<snipp>
zerg.hydra
configs:
partitions: 3
partition 0
leader: 1 (192.168.0.153:9092)
replicas: 1 (192.168.0.153:9092), 2 (192.168.0.153:9093)
isr: 1 (192.168.0.153:9092), 2 (192.168.0.153:9093)
partition 1
leader: 2 (192.168.0.153:9093)
replicas: 2 (192.168.0.153:9093), 3 (192.168.0.153:9094)
isr: 2 (192.168.0.153:9093), 3 (192.168.0.153:9094)
partition 2
leader: 3 (192.168.0.153:9094)
replicas: 3 (192.168.0.153:9094), 1 (192.168.0.153:9092)
isr: 3 (192.168.0.153:9094), 1 (192.168.0.153:9092)
<snipp>
```

#### 启动生产者

采样`sync模式`启动一个生产者

```
bin/kafka-console-producer.sh --broker-list localhost:9092,localhos
t:9093,localhost:9094 --sync \
--topic zerg.hyd
```

启动后的输出：

```
[...] INFO Verifying properties (kafka.utils.VerifiableProperties)
[...] INFO Property broker.list is overridden to localhost:9092,loc
alhost:9093,localhost:9094 (...)
[...] INFO Property compression.codec is overridden to 0 (kafka.uti
ls.VerifiableProperties)
[...] INFO Property key.serializer.class is overridden to kafka.ser
ializer.StringEncoder (...)
[...] INFO Property producer.type is overridden to sync (kafka.util
s.VerifiableProperties)
[...] INFO Property queue.buffering.max.messages is overridden to 1
0000 (...)
[...] INFO Property queue.buffering.max.ms is overridden to 1000 (k
afka.utils.VerifiableProperties)
[...] INFO Property queue.enqueue.timeout.ms is overridden to 0 (ka
fka.utils.VerifiableProperties)
[...] INFO Property request.required.acks is overridden to 0 (kafk
a.utils.VerifiableProperties)
[...] INFO Property request.timeout.ms is overridden to 1500 (kafk
a.utils.VerifiableProperties)
[...] INFO Property send.buffer.bytes is overridden to 102400 (kafk
a.utils.VerifiableProperties)
[...] INFO Property serializer.class is overridden to kafka.seriali
zer.StringEncoder (...)
```

启动完毕后，就可以向kafka集群发送消息：

```
Hello, world!
Rock: Nerf Paper. Scissors is fi
```

#### 启动消费者
启动一个消费者终端读取topic中的信息

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic ze
rg.hydra --from-beginning
```
成功输出如下

```
Hello, world!
Rock: Nerf Paper. Scissors is fine.
```