## kafka初体验


kafka 是支持 一个 consumer 监听多个 topic 的。

这就导致 如果业务上不需要 某个 consumer 去监听多个topic 就要保证该 consumer 的 groupId 唯一。

也就是说 topic A 的一个 consumer groupId 是 ：test
那对于topic B 的一个 consumer groupId 就不能是 test （除非你需要 test 这组 consumer 监听 topic A 和 topic B）；

不然就会导致 zk 频繁的 reblance

![](https://tva1.sinaimg.cn/large/006tNbRwly1gb1v3v2d5mj316t0u00ui.jpg)


对应同一个topic可以任意添加消费者，如果消费者之间的group相同就是在一个组里面， 在一个组的消费者之间是竞争关系，只有一个消费者可以消费这个topic的一条消息。

如果有多个 group 那group之间是平级关系，每一个 group都可以接受到消息。

kafka 默认存放7天的临时数据，如果遇到磁盘空间小，存放数据量大，可以设置缩短这个时间。

[kafka频繁reblance](https://olnrao.wordpress.com/2015/05/15/apache-kafka-case-of-mysterious-rebalances/)

#### 1. 什么是 Kafka

> Kafka 是一个分布式流式平台，它有三个关键能力

1、订阅发布记录流，它类似于企业中的消息队列或企业消息传递系统
2、以容错的方式存储记录流
3、实时记录流

#### 2. kafka 核心 APIs

>Kafka 有四个核心API，它们分别是

1、Producer API，它允许应用程序向一个或多个 topics 上发送消息记录
2、Consumer API，允许应用程序订阅一个或多个 topics 并处理为其生成的记录流
3、Streams API，它允许应用程序作为流处理器，从一个或多个主题中消费输入流并为其生成输出流，有效的将输入流转换为输出流。
4、Connector API，它允许构建和运行将 Kafka 主题连接到现有应用程序或数据系统的可用生产者和消费者。例如，关系数据库的连接器可能会捕获对表的所有更改

![](https://tva1.sinaimg.cn/large/006tNbRwly1gb1zi8sv7pj31cc0twwfo.jpg)

#### 3. kafka 重要概念

##### topic
>topic 被称为主题，在 kafka 中，使用一个类别属性来划分消息的所属类，划分消息的这个类称为 topic。topic 相当于消息的分配标签，是一个逻辑概念。主题好比是数据库的表，或者文件系统中的文件夹。

##### partition
>partition 译为分区，topic 中的消息被分割为一个或多个的 partition，它是一个物理概念，对应到系统上的就是一个或若干个目录，一个分区就是一个 提交日志。消息以追加的形式写入分区，先后以顺序的方式读取。分区可以分布在不同的服务器上，也就是说，一个主题可以跨越多个服务器，以此来提供比单个服务器更强大的性能。

![](https://tva1.sinaimg.cn/large/006tNbRwly1gb1znjt2jrj30ti0f6gm2.jpg)

==注意：由于一个主题包含无数个分区，因此无法保证在整个 topic 中有序，但是单 partition 分区可以保证有序。消息被迫加写入每个分区的尾部。Kafka 通过分区来实现数据冗余和伸缩性==

##### segment

>Segment 被译为段，将 Partition 进一步细分为若干个 segment，每个 segment 文件的大小相等。

##### borker

>Kafka 集群包含一个或多个服务器，每个 Kafka 中服务器被称为 broker。broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。broker 为消费者提供服务，对读取分区的请求作出响应，返回已经提交到磁盘上的消息。

>broker 是集群的组成部分，每个集群中都会有一个 broker 同时充当了 集群控制器(Leader)的角色，它是由集群中的活跃成员选举出来的。每个集群中的成员都有可能充当 Leader，Leader 负责管理工作，包括将分区分配给 broker 和监控 broker。集群中，一个分区从属于一个 Leader，但是一个分区可以分配给多个 broker（非Leader），这时候会发生分区复制。这种复制的机制为分区提供了消息冗余，如果一个 broker 失效，那么其他活跃用户会重新选举一个 Leader 接管。

![](https://tva1.sinaimg.cn/large/006tNbRwly1gb1zxjta85j30u00jl3ze.jpg)

##### producer
>生产者，即消息的发布者，其会将某 topic 的消息发布到相应的 partition 中。生产者在默认情况下把消息均衡地分布到主题的所有分区上，而并不关心特定消息会被写到哪个分区。不过，在某些情况下，生产者会把消息直接写到指定的分区。

##### consumer
>消费者，即消息的使用者，一个消费者可以消费多个 topic 的消息，对于某一个 topic 的消息，其只会消费同一个 partition 中的消息

![](https://tva1.sinaimg.cn/large/006tNbRwly1gb1zx1zetbj30u00ao74r.jpg)

#### 2. Kafka 的应用场景

> Kafka 可以建立流数据管道，可靠性的在系统或应用之间获取数据。建立流式应用传输和响应数据。

1、作为消息系统
2、作为存储系统
3、作为流处理器

##### 2.1 Kafka 作为消息系统

>Kafka 作为消息系统，它有三个基本组件

![](https://tva1.sinaimg.cn/large/006tNbRwly1gb1zbwam22j31ha0k20tg.jpg)

+ Producer : 发布消息的客户端
+ Broker：一个从生产者接受并存储消息的客户端
+ Consumer : 消费者从 Broker 中读取消息

资料：
[Kafka Producer](https://mp.weixin.qq.com/s/mpUQhmrzaBT0GxaoOcf0nw)
[Kafka Counsumer](https://mp.weixin.qq.com/s/N87cD0zMc211TwxXu_-wkA)
[kafka](https://mp.weixin.qq.com/s/UULQ8V6RgRaSmoH64TEcFQ)
[kafka控制器](https://mp.weixin.qq.com/s/6IXlnQjuup366JQYlTsJ9w)
[kafka副本机制和请求过程](https://mp.weixin.qq.com/s/cWF1m3ihwsix-jnGpyAjOg)
[kafka](https://mp.weixin.qq.com/s/ddTKYHWK8U5VJMWmqmi3vA)