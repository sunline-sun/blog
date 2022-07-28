#### HBase出现背景

关系型数据库处理海量数据能力的欠缺和僵硬的设计约束，为了解决关系型数据库存在的问题，从Google的BigTable论文开始，NoSQL被提出来，HBase是这一类NoSQL的代表



#### HBase可伸缩架构

HBase的伸缩性主要依赖其可分裂的HRegion及可伸缩的分布式文件系统HDFS实现。数据以HRegion为单位进行管理，应用程序需要操作数据，需要先找到HRegion，然后将读写请求提交给HRegion。

![](https://static001.geekbang.org/resource/image/9f/f7/9f4220274ef0a6bcf253e8d012a6d4f7.png?wh=1396*454)

- HRegionServer：HRegionServer是物理服务器，每个HRegionServer上可以启动多个HRegion实例。当一个HRegion中写入数据达到阈值后，一个HRegion会分裂为两个HRegion，并将HRegion在整个集群中迁移，使HRegionServer负载均衡。
- HRegion：数据的基本操作单位。每个HRegion中存储一段key值区间【key1，key2】的数据
- HMaster：存储HRegion的key值区间、所在HRegionServer地址、访问端口号等信息。为了保证高可用，HBase会启动多个HMaster，并通过Zookeeper选举一个主服务器。
- HFile：数据文件，HRegion会把数据存储在多个HFile格式的文件中，这些文件使用HDFS进行存储，在整个集群内分布并高可用。



##### 访问数据时序图

![img](https://static001.geekbang.org/resource/image/9f/ab/9fd982205b06ecd43053202da2ae08ab.png?wh=1406*600)

#### HBase可扩展数据模型

传统关系数据库设计表的时候需要制定字段名称、数据类型，无法随时扩展。所以许多NoSQL数据使用列族（ColumnFamily）解决。下面是面向列族的稀疏矩阵存储格式。

![img](https://static001.geekbang.org/resource/image/74/6f/74b3aac940abae8a571cc94f2226656f.png?wh=1188*210)

在创建表的时候，只需要指定列族名称，无需指定字段，在写入数据时候在指定字段，通过这种方式，数据表可以包含数百万字段，可以随意扩展应用程序的数据结构。查询时候也可以通过字段名称和值查询。



#### HBase的高性能存储

机械硬盘的特点是，连续读写很快，随机读写很慢。因为机器磁盘靠电机驱动访问磁盘上的数据，电机要将磁头落在数据所在的磁道上，这个过程需要较长的寻址时间，如果不是连续存储，磁头需要不停的移动，浪费大量的时间。

所以为了提高数据写入速度，HBase使用LSM树的数据结构进行数据存储。LSM全名是Log Structed Merge Tree，Log结构合并树，数据写入时候以Log方式连续写入，然后异步对磁盘上的多个LSM树进行合并。

![img](https://static001.geekbang.org/resource/image/5f/3b/5fbd17a9c0b9f1a10347a4473d00ad3b.jpg?wh=4411*1911)

- LSM树可以看做是一个N阶合并树，数据写操作都在内存中进行，这些数据在内存中是一颗排序树
- 当数据量超过阈值后，会将内存中的排序树和磁盘上最新的排序树合并。当这颗排序树也超过阈值了，会和磁盘下一级的排序树合并
- 读操作时，总在内存中的排序树开始搜索，如果没有在依次从磁盘上的排序树搜索
- 写操作时，会先写日志用作断电恢复等问题，然后直接更新内存中排序树。
- LSM对于写操作为主，读操作集中在最近写入的数据上的场景，可以极大减少磁盘的访问次数，加快访问速度。
