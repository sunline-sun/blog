#### Spark出现背景

- 为了提高用户体验，UC Berkeley的AMP Lab开发的Spark推出后，拥有更快的执行速度和更友好的编程接口，并且支持Yarn和HDFS，解决了MapReduce执行速度和编程复杂度的问题。所以开始迅速抢占MapReduce的市场，成为主流的大数据计算框架。
- Spark的性能在逻辑回归机器学习性能上，Spark比MapReduce快了100多倍



#### Spark编程模型

```scala
### wordCount 在spark中的实现
val textFile = sc.textFile("hdfs://...")
val counts = textFile.flatMap(line => line.split(" "))
                 .map(word => (word, 1))
                 .reduceByKey(_ + _)
counts.saveAsTextFile("hdfs://...")
```



- Spark的编程模型是RDD，RDD是弹性数据集（Resilient Distributed Datasets）的缩写。
- RDD在Spark中即是一个逻辑概念（代码中的RDD对象），又是一个物理概念（物理上的RDD分片）
- RDD定义函数分为两种，一种是转换函数，返回值是RDD，另一种是执行函数，不在返回RDD
- RDD定义了很多转换函数，如计算（map）、过滤（filter）、合并数据集（union）、聚合（reduceByKey）、链接数据集（join）、分组（groupByKey）等十几个函数。
- 转换操作生成的RDD不一定会出现新的分片，如map、filter等，是在当前map处理数据的，像reduceByKey这种转换需要聚合操作，才会产生新的RDD分片，这种特性也被称作惰性设计。



#### 编程模型和MapReduce区别

- MapReduce：将计算分为Map和Reduce两个阶段，编程中重点在于两个阶段的实现方式和输入输出，是面向过程的大数据计算
- Spark：针对数据进行变成，将大数据集合抽象为RDD对象，在RDD的上进行计算处理，得到新的RDD，直到得到最后结果。编程中的重点在于RDD对象。是面向对象的大数据计算。



#### Spark生态体系

- 支持SQL语句的Spark SQL
- 支持流计算的Spark Streaming
- 支持机器学习的MLlib
- 支持图计算的GraphX

![img](https://static001.geekbang.org/resource/image/38/0f/3894be10797c657af3a54bc278ab780f.png?wh=620*308)



#### Spark计算阶段

![img](https://static001.geekbang.org/resource/image/c8/db/c8cf515c664b478e51058565e0d4a8db.png?wh=1296*712)

- 图里每个RDD里包含多个小块，每个小块代表RDD的一个分片



```
#### 上面的伪代码
rddB = rddA.groupBy(key)
rddD = rddC.map(func)
rddF = rddD.union(rddE)
rddG = rddB.join(rddF)
```



- Spark计算阶段的核心是DAG（有向无环图）
- Spark会根据应用的复杂程度，分割成更多的计算阶段（stage），这些计算阶段组成了DAG，建立了依赖关系，DAGScheduler组件负责Spark应用DAG的生成和管理，根据代码生成DAG，然后将程序分发到分布式计算集群。
  - stage划分依据是shuffle，而不是转换函数的类型，shuffle和MapReduce的shuffle一样，都是将不同数据分片分区传输合并使用。
  - 依赖关系分为两种，不需要进行shuffle的，如B -> G，称为窄依赖，需要进行shuffle的，如F -》G，称为宽依赖。
- 根据DAG的依赖关系顺序执行各个计算阶段，根据每个阶段要处理的数据量生成相应的任务集合（TaskSet），完成分布式计算。



#### Spark比MapReduce计算块的原因

- 某些机器学习算法需要进行大量迭代计算，产生数万个计算阶段，Spark可以在一个应用中处理完成，MapReduce需要启动数万个应用。所以Spark可以有效减少对HDFS的访问，减少作业的调度执行次数，执行速度更快
  - 因为Spark将前一个reduce和后一个map连接了起来，所以可以在一个应用中处理完成。
- MapReduce主要使用磁盘存储shuffle过程中的数据，Spark优先使用缓存，包括RDD数据



#### Spark的作业管理

![img](https://static001.geekbang.org/resource/image/2b/d0/2bf9e431bbd543165588a111513567d0.png?wh=668*188)

- 任务（task）：Spark为RDD每个数据分片创建一个任务处理，一个计算阶段包含多个计算任务
- job：DAGScheduler遇到action函数（调用后不返回RDD，如count()、saveAsTextFile()等），会生成一个job
- 上图中，横轴是时间，纵轴是任务，两条粗黑线之间是一个作业，两条细线之间是一个计算阶段，一个作业至少包含一个计算阶段。





#### Spark的执行过程

spark支持Standalone、Yarn、Mesos、Kubernetes等多种部署方式

![img](https://static001.geekbang.org/resource/image/16/db/164e9460133d7744d0315a876e7b6fdb.png?wh=596*286)

1. Spark应用程序启动在自己的JVM进程里，即Driver进程，启动后调用SparkContext初始化执行配置和输入数据
2. SparkContext启动DAGScheduler构造执行的DAG图，切分成最小的执行单位也就是计算任务。
3. Driver向Cluster Manger请求计算资源，用于DAG的分布式计算。
4. Cluster Manger收到请求后，将Driver的主机地址等信息通知给集群的所有计算节点Worker
5. Worker收到消息后，根据Driver主机地址，跟Driver通信并注册，根据自己的空闲资源向Driver通报自己可以领取的任务数。
6. Driver根据DAG图开始向注册的Worker分配任务。
7. Worker收到任务后，启动Executor进程开始执行任务。
8. Executor检查自己是否有Driver执行代码，没有就从Driver下载，通过Java反射加载后开始执行。
