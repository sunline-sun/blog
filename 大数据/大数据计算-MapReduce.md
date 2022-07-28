### MapReduce

MapReduce即是一个编程模型，又是一个计算框架，开发人员必须基于MapReduce编程模型开发，然后通过MapReduce计算框架分发到Hadoop集群中运行。

#### 编程模型

- 包含两个过程：Map和Reduce，输入一对<key,value>值，经过map计算输出一对<key,value>，然后将相同key合并，形成<key,value集合>,再将这个输入给reduce，经过计算输出<key,value>对
- 包含关系代数运算、矩阵运算

以WordCount为例，看看用普通的Python编程和MapReduce编程的区别，数据量大了之后，肯定不能用Python将所有数据加载到内存运算

```python

# 文本前期处理
strl_ist = str.replace('\n', '').lower().split(' ')
count_dict = {}
# 如果字典里有该单词则加1，否则添加入字典
for str in strl_ist:
if str in count_dict.keys():
    count_dict[str] = count_dict[str] + 1
    else:
        count_dict[str] = 1
```

```mapReduce

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
}
```

以上MapReduce程序中，入参格式为<1,aaa>，value是要统计的单词，map函数将文本中的单词提取出来，输出格式为<aaa,1>，key为单词，value为1，代表计数1，MapReduce计算框架将map集合起来，相同key的放在一起，形成<aaa,1,1,1>格式，作为value的输入参数，value根据集合里的1求和，统计单词的数量，输出<aaa,3>格式



#### 计算框架

![img](https://static001.geekbang.org/resource/image/f3/9c/f3a2faf9327fe3f086ec2c7eb4cd229c.png?wh=1434*300)





#### 计算中的两个主要问题

- 如何分发
- 如何合并



#### 分发计算过程

- 大数据应用进程：启动MapReduce的主入口，也就是用户自己写的程序，指定Map和Reduce类，提交作业给Hadoop集群，也就是JobTracker进程。
- JobTracker进程：根据要处理的数据量，命令TaskTracker进程启动相应数量的Map和Reduce程序，并管理整个作业的生命周期的任务调度和监控
- TaskTracker进程：负责启动和管理Map进程和Reduce进程，通常和HDFS的DataNode进程启动在同一个服务器上，因为每个数据块都有对应的Map程序。
- JobTracker和TaskTracker是一多主从关系，JobTracker是主服务器，负责管理，TaskTracker是从进程，负责执行，大部分大数据框架都是主从架构



![](https://static001.geekbang.org/resource/image/2d/27/2df4e1976fd8a6ac4a46047d85261027.png?wh=1368*802)

1. 应用进程JobClient将用户作业Jar包存储在HDFS中
2. 应用程序提交job作业给JobTracker
3. JobTracker根据作业调度策略创建JobInProcess树，每个作业会有自己的JobInProcess树
4. JobInProcess根据输入数据的分片数目（通常是数据块的数目）和设置的Reduce数目创建响应数量的TaskInProcess
5. TaskTracker进程和JobTracker进程定时通讯
6. 如果TaskTracker有空闲的计算资源，JobTracker会给他分配任务。分配的时候，会根据TaskTracker的服务器名称匹配同一个机器上的数据块计算任务给他，也就是计算任务处理本机上的数据，也就是一开始说的移动计算。
7. TaskTracker收到任务后，根据任务类型（map和Reduce）和任务参数（Jar包路径、输入数据文件路径、要处理的数据再文件中的起始位置和偏移量、数据块多个备份的DataNode主机名等），启动相应的Map和Reduce进程。
8. Map或者Reduce进程启动后，检查本地是否有要执行的Jar包文件，如果没有，就去HDFS上下载，然后加载Map或者Reduce代码开始执行
9. 如果是Map进程，从HDFS读取数据，如果是Reduce进程，将结果数据写出到HDFS



#### 合并计算过程

![img](https://static001.geekbang.org/resource/image/d6/c7/d64daa9a621c1d423d4a1c13054396c7.png?wh=1316*772)



1. 每个Map任务的计算结果会写入到本地文件系统
2. MapReduce计算框架启动shuffle过程，在Map任务进程调用一个Partitioner接口，对Map产生的<key,value>进行Reduce分区选择
   - 默认Partitioner是用Key的哈希值对Reduce任务数量取模，使相同的key一定落下相同的Reduce任务ID上
3. 分区完成后，通过HTTP请求发送给对应的Reduce进程
4. Reduce任务进程对收到的<key,value>进行排序和合并，相同key的放在一起，组成<key,value集合>传递给Reduce执行

