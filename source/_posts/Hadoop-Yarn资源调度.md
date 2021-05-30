---
title: Hadoop-Yarn资源调度
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-17 20:54:46
password:
summary: Yarn资源调度，这块会跟前面说的所有的东西联系起来
tags:
- Hadoop-Yarn
categories:
- 大数据
---

# Hadoop

##  Yarn资源调度器

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。

### Yarn基本架构

YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等组件构成，如图:

![](66.png)

ResourceManager：整个集群老大
NodeManager：某个节点老大
ApplicationMaster：job老大
Container：虚拟化资源分配

### Yarn工作机制

1．Yarn运行机制，如图：

![](67.png)

2．工作机制详解

（1）MR程序提交到客户端所在的节点。

（2）YarnRunner向ResourceManager申请一个Application。

（3）RM将该应用程序的资源路径返回给YarnRunner。

（4）该程序将运行所需资源提交到HDFS上。

（5）程序资源提交完毕后，申请运行mrAppMaster。

（6）RM将用户的请求初始化成一个Task。

（7）其中一个NodeManager领取到Task任务。

（8）该NodeManager创建容器Container，并产生MRAppmaster。

（9）Container从HDFS上拷贝资源到本地。

（10）MRAppmaster向RM 申请运行MapTask资源。

（11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

（12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

（13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

（14）ReduceTask向MapTask获取相应分区的数据。

（15）程序运行完毕后，MR会向RM申请注销自己。

### 作业提交全过程

1．作业提交过程之YARN，如图：

![](67.png)

Map过程：read（读取）——>map（分）——>collect（收集）——>spill（溢写）——>merge（合并）

Reduce过程：copy（拷贝map过程的数据）——>merge+sort（归并排序）——>reduce

作业提交全过程详解

（1）作业提交

第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。

第2步：Client向RM申请一个作业id。

第3步：RM给Client返回该job资源的提交路径和作业id。

第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。

第5步：Client提交完资源后，向RM申请运行MrAppMaster。

（2）作业初始化

第6步：当RM收到Client的请求后，将该job添加到容量调度器中。

第7步：某一个空闲的NM领取到该Job。

第8步：该NM创建Container，并产生MRAppmaster。

第9步：下载Client提交的资源到本地。

（3）任务分配

第10步：MrAppMaster向RM申请运行多个MapTask任务资源。

第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

（4）任务运行

第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。

第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。

第14步：ReduceTask向MapTask获取相应分区的数据。

第15步：程序运行完毕后，MR会向RM申请注销自己。

（5）进度和状态更新

YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

（6）作业完成

除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

2．作业提交过程之MapReduce，如图：

![](68.png)

其中HDFS文件操作中就牵扯到了NameNode和Secondary NameNode相等问题（日志+2NN=NN）

### 资源调度器（作业提交过程中任务队列）

目前，Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler和Fair Scheduler。Hadoop2.7.2默认的资源调度器是Capacity Scheduler。

[yarn-default.xml]

```xml
    <property>
        <description>
            The class to use as the resource scheduler.</description>
        <name>yarn.resourcemanager.scheduler.class</name>
        <value>
            org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler
        </value>
    </property>
```

1．先进先出调度器（FIFO）

![](69.png)

其中分配多少个task，得看现在可用资源有多少。其中可能一下分配很多个task。

2．容量调度器（Capacity Scheduler）

![](70.png)

多个FIFO调度器的总和

3．公平调度器（Fair Scheduler）

![](71.png)

容量调度器中，作业并不是公平享有，得按一定规则排好序，然后按优先级来获取资源。

如果机器性能高，想要并发度，可以采用第三种；若机器性能差点，还想要并发度，可以采用第二种；第一种完全没有并发度。

### 任务的推测执行

1．作业完成时间取决于最慢的任务完成时间

一个作业由若干个Map任务和Reduce任务构成。因硬件老化、软件Bug等，某些任务可能运行非常慢。

2．推测执行机制

发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果。

3．执行推测任务的前提条件

（1）每个Task只能有一个备份任务

（2）当前Job已完成的Task必须不小于0.05（5%）

（3）开启推测执行参数设置。mapred-site.xml文件中默认是打开的。

```xml
<property>
  	<name>mapreduce.map.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
</property>

<property>
  	<name>mapreduce.reduce.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
</property>
```

4．不能启用推测执行机制情况

（1）任务间存在严重的负载倾斜；

（2）特殊任务，比如任务向数据库中写数据。

5．算法原理，如图

![](72.png)

# 待续...

Hadoop先告一段落了，准备再学习下Spark，希望将要有好事发生~

> 希望屁屁在剩下的这一年中不会太忙，加班越来越少，工资越来越高
> 希望接下来一切顺利，好事发生~