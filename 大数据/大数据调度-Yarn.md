#### Yarn产生的背景

在MapReduce架构中，服务器集群资源调度管理和MapReduce执行过程耦合在一起，如果想在当前集群中运行其他计算任务，比如Spark或者Storm，就无法统一使用集群中的资源了。

早期，大数据技术只有Hadoop一家，这个缺点不明显，后来随着发展，各种新的计算框架出现，不可能为每个计算框架部署一套集群，所以需要把资源管理和计算框架分开。



#### Yarn定义

Yarn是“Yet Another Resource Negotiator"缩写，代表另一种资源调度器。

Yarn的出现，使Hadoop从一个单一的大数据计算引擎，变成了一个集存储、计算、资源管理为一体的完成大数据平台。



#### Yarn架构

![img](https://static001.geekbang.org/resource/image/af/b1/af90905013e5869f598c163c09d718b1.jpg?wh=2844*1712)



- 资源管理器（Resource Manager）：负责整个集群的资源调度工作，部署在独立的服务器上
  - 调度器：调度器是一个资源分配算法，根据client提交的资源申请和当前服务器集群的资源状况进行资源分配。内置几种调度算法，包括Fair Scheduler、Capacity Scheduler等，也可以开发自己的调度算法。
  - 应用程序管理器：负责应用程序的提交、监控应用程序运行状态，
- 节点管理器（Node Manager）：负责具体服务器上的资源和任务管理，部署在每个服务器上，通常就在HDFS的DataNode在一起
  - 容器：Yarn进行资源分配的单位，每个容器中包含一定的内存、CPU，由Node Manager进行管理，会监控本节点的容器并上报运行情况给Resource Manager进程。
  - ApplicationMaster：每个应用程序有自己的ApplicationMaster，运行在容器中。应用程序启动后，由ApplicationMaster根据应用程序的资源需求向ResourceManager进程申请容器资源，的到容器后，会分发自己的应用程序代码到容器上启动，开始分布式计算。



#### Yarn工作流程

1. 向Yarn提交应用程序，包括MapReduce ApplicationMaster、MapReduce程序、MapReduce Application启动命令
2. ResourceManager进程和NodeManager进程通信，根据集群资源，为用户程序分配第一个容器，并将MapReduce ApplicationMaster分发到这个容器上面，在容器中启动。
3. MapReduce ApplicationMaster启动后向ResourceManager注册，为自己的应用程序申请容器资源
4. MapReduce ApplicationMaster申请到需要的容器后，和容器所在的NodeManager进行通讯，将用户MapReduce程序分发到NodeManager所在服务器上，并在容器中运行，也就是Map或者Reduce任务。
5. Map或者Reduce任务执行期间，和MapReduce ApplicationMaster通信，汇报自己运行情况
6. 任务执行结束后，MapReduce ApplicationMaster向ResourceManager进程注销并释放所有的容器资源。