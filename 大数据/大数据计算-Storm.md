#### 流计算的产生背景

像MapReduce、Spark这种大数据计算框架，是针对HDFS上大规模数据每天或者定时处理一次，以批为单位，是批处理计算。

但是像需要实时计算处理的场景，如摄像头实时视频分析、淘宝实时订单数据，需要实时处理，这种计算方式称为流处理计算，常见框架有strom、spark Streaming、Flink



#### 无计算框架处理流计算方案

可以使用消息队列实现大数据的实时处理，当然，如果处理复杂，则需要多个消息队列，由消息队列负责数据的流转，处理逻辑即是前面消息的消费者，也是后面消息的生产者。

但是这样存在一个问题，就是这样的系统是根据不同需求开发的，每次新的需求需要重新开发类似的系统，因为处理逻辑不同了。

![img](https://static001.geekbang.org/resource/image/19/31/199c65da1a9dfae48f42c32f6a82c831.png?wh=1236*312)



#### Storm

在消息队列处理方案的背景下，Storm出现了，让我们无需关注数据流转、消息的处理和消费，只要编程开发好数据处理逻辑和数据源的逻辑，以及他们之间的拓扑关系，提交到Storm就可以了

![img](https://static001.geekbang.org/resource/image/78/5b/780899b3fda0ea39acbdfb9545fbc55b.png?wh=664*314)



#### Storm架构

storm和Hadoop一样，也是主从架构。nimbus和supervisor之间通过Zookeeper完成任务分配、心跳检测等操作。

![img](https://static001.geekbang.org/resource/image/d3/8a/d33aa8765ad381824fd9818f93074a8a.png?wh=1198*762)

- nimbus：集群的Master，负责集群管理、任务分配等。
- supervisor：集群的Slave，真正完成计算的地方，每个supervisor会启动多个worker进程
- worker：工作进程，每个worker会运行多个task
- task：实际工作任务，就是spout或者bolt