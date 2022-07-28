#### Hive产生背景

- MapReduce的出现大大简化了编程开发的难度，但是对于经常使用大数据计算的人来说，通常是使用SQL进行数据分析和统计，如果每次统计都开发MapReduce程序，成本比较高
- 所以Hive出现了，他主要是自动将SQL生成MapReduce代码，然后提交给Hadoop执行。



#### Hive架构

![img](https://static001.geekbang.org/resource/image/26/ea/26287cac9a9cfa3874a680fdbcd795ea.jpg?wh=720*383)

- 通过hive的client提交SQL命令
- Driver判断SQL指令是DDL还是DQL，如果是DDL会将数据表的信息记录在Metastore元数据组件中。
- 如果是DQL查询语句，会将SQL提交给自己的编译器Compiler进行语法分析、语法解析、语法优化等操作，生成一个MapReduce执行计划，然后根据执行计划生成MapReduce作业
  - Hive内置了很多函数，执行计划就是根据SQL语句生成这些函数的DAG（有向无环图），然后封装进MapReduce的map和reduce函数中。
- 提交MapReduce作业给Hadoop的MapReduce计算框架执行作业。



#### 其他的SQL引擎

- Cloudera开发的Impala，运行在HDFS上的MPP架构的SQL引擎
- Spark自己的SQL引擎Shark，也就是后来的Spark SQL
- Hive退出了Hive on Spark
- Saleforce推出了执行在HBase上的SQL引擎Phoenix