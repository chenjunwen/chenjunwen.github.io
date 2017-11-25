---
title: Kafka-windows
date: 2017-11-25 16:27:03
tags:
---

# 1.序言

>Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群来提供实时的消费。

>Kafka[1]  是一种高吞吐量[2]  的分布式发布订阅消息系统，有如下特性：
通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。
高吞吐量[2]  ：即使是非常普通的硬件Kafka也可以支持每秒数百万[2]  的消息。
支持通过Kafka服务器和消费机集群来分区消息。
支持Hadoop并行数据加载

>Kafka必须和zookeeper一起使用

# 2.Kafka相关术语介绍
- Broker
    
    Kafka集群包含一个或多个服务器，这种服务器被称为broker[5] 
- Topic

    每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）
- Partition
    
    Partition是物理上的概念，每个Topic包含一个或多个Partition.
- Producer
    
    负责发布消息到Kafka broker
- Consumer

    消息消费者，向Kafka broker读取消息的客户端。
- Consumer Group

    每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）
# 三、安装与运行Kafka
## 下载
http://kafka.apache.org/downloads.html
注意要下载二进制版本的
![image](http://img.blog.csdn.net/20160903095658191)
下载后解压到任意一个目录，笔者的是D:\kafka_2.11-0.10.0.1
1. 进入Kafka配置目录，D:\kafka_2.11-0.10.0.1
2. 编辑文件“server.properties”
3. 找到并编辑log.dirs=D:\kafka_2.11-0.10.0.1\kafka-log,这里的目录自己修改成自己喜欢的
4. 找到并编辑zookeeper.connect=localhost:2181。表示本地运行
5. Kafka会按照默认，在9092端口上运行，并连接zookeeper的默认端口：2181。
## 运行：
重要：请确保在启动Kafka服务器前,如果报错的话就重启电脑，Zookeeper实例已经准备好并开始运行。

1. 进入Kafka安装目录D:\kafka_2.11-0.10.0.1
2. 按下Shift+右键，选择“打开命令窗口”选项，打开命令行。
3. 现在输入
>     .\bin\windows\kafka-server-start.bat .\config\server.properties   


## [原文地址](http://blog.csdn.net/evankaka/article/details/52421314)

# 四、测试
上面的Zookeeper和kafka一直打开
- （1）、创建主题
1. 进入Kafka安装目录D:\kafka_2.11-0.10.0.1
2. 按下Shift+右键，选择“打开命令窗口”选项，打开命令行。
3. 现在输入
    
>.\bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic linlin

#### 解释
- --replication-factor 2   #复制两份
- --partitions 1 #创建1个分区
- --topic #主题为shuaige

==不要关闭窗口==

- （2）创建生产者
1. 进入Kafka安装目录D:\kafka_2.11-0.10.0.1
2. 按下Shift+右键，选择“打开命令窗口”选项，打开命令行。
3. 现在输入
>.\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic linlin

- （3）创建消费者
1. 进入Kafka安装目录D:\kafka_2.11-0.10.0.1
2. 按下Shift+右键，选择“打开命令窗口”选项，打开命令行。
3. 现在输入
>.\bin\windows\kafka-console-consumer.bat --zookeeper localhost:2181 --topic linlin


- （4）其他命令
> 查看topic


    ./kafka-topics.sh --list --zookeeper localhost:12181
    #就会显示我们创建的所有topic

>查看topic状态

    /kafka-topics.sh --describe --zookeeper localhost:12181 --topic shuaige
    #下面是显示信息
    Topic:ssports    PartitionCount:1    ReplicationFactor:2    Configs:
        Topic: shuaige    Partition: 0    Leader: 1    Replicas: 0,1    Isr: 1
    #分区为为1  复制因子为2   他的  shuaige的分区为0 
    #Replicas: 0,1   复制的为0，1
>更多请看官方文档：http://kafka.apache.org/documentation.html


>zookeeper 和 kafka 集群搭建: http://blog.csdn.net/my_bai/article/details/68490632