---
title: Hadoop-HDFS-DataNode相关（系列十二）
top: false
cover: true
toc: true
mathjax: true
date: 2020-09-01 20:49:39
password:
summary: DataNode相关（面试开发重点）
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## DataNode（面试开发重点）

### DataNode工作机制

![](1.png)

一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

1）DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。

2）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令（如复制块数据到另一台机器，或删除某个数据块）。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

3）集群运行中可以安全加入和退出一些机器。

### 数据完整性

**思考：如果电脑磁盘里面存储的数据是控制高铁信号灯的红灯信号（1）和绿灯信号（0），但是存储该数据的磁盘坏了，一直显示是绿灯，是否很危险？同理DataNode节点上的数据损坏了，却没有发现，是否也很危险，那么如何解决呢？**

如下是DataNode节点保证数据完整性的方法。

1）当DataNode读取Block的时候，它会计算CheckSum。

2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。

3）Client读取其他DataNode上的Block。

4）DataNode在其文件创建后周期验证CheckSum，如图：

![](2.png)

### 掉线时限参数设置

![](4.png)

需要注意的是hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒。

```xml
<property>
    <name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>300000</value>
</property>
<property>
    <name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>
```

### 服役新数据节点

0. 需求
随着公司业务的增长，数据量越来越大，原有的数据节点的容量已经不能满足存储数据的需求，需要在原有集群基础上动态添加新的数据节点。

1.	环境准备

（1）在hadoop104主机上再克隆一台hadoop105主机

（2）修改IP地址和主机名称

（3）删除原来HDFS文件系统留存的文件（/opt/module/hadoop-2.7.2/data和log）

（4）source一下配置文件

```shell
[user_test@hadoop105 hadoop-2.7.2]$ source /etc/profile
```

2.	服役新节点具体步骤

（1）直接启动DataNode，即可关联到集群

```shell
[user_test@hadoop105 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start datanode

[user_test@hadoop105 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager
```

![](5.png)

（2）在hadoop105上上传文件

```shell
[user_test@hadoop105 hadoop-2.7.2]$ hadoop fs -put /opt/module/hadoop-2.7.2/LICENSE.txt /
```

（3）如果数据不均衡，可以用命令实现集群的再平衡

```shell
[user_test@hadoop102 sbin]$ ./start-balancer.sh
——>
starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out
Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved
```

### 退役旧数据节点

#### 添加白名单

添加到白名单的主机节点，都允许访问NameNode，不在白名单的主机节点，都会被退出。

配置白名单的具体步骤如下：

（1）在NameNode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts文件，添加以下内容

```shell
[user_test@hadoop102 hadoop]$ pwd
——>
/opt/module/hadoop-2.7.2/etc/hadoop

[user_test@hadoop102 hadoop]$ touch dfs.hosts

[user_test@hadoop102 hadoop]$ vi dfs.hosts
```

添加如下主机名称（不添加hadoop105）
hadoop102
hadoop103
hadoop104

（2）在NameNode的hdfs-site.xml配置文件中增加dfs.hosts属性

```xml
<property>
	<name>dfs.hosts</name>
	<value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts</value>
</property>
```

（3）配置文件分发

```shell
[user_test@hadoop102 hadoop]$ xsync hdfs-site.xml
```

（4）刷新NameNode

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes
——>
Refresh nodes successful
```

（5）更新ResourceManager节点

```shell
[user_test@hadoop103 hadoop-2.7.2]$ yarn rmadmin -refreshNodes
——>
17/06/24 14:17:11 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033
```

（6）在web浏览器上查看

![](6.png)

4.	如果数据不均衡，可以用命令实现集群的再平衡

```shell
[user_test@hadoop102 sbin]$ ./start-balancer.sh
——>
starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out
Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved
```

#### 黑名单退役

**在做以下操作时，要注意退回到上一节操作之前状态**

在黑名单上面的主机都会被强制退出。

1.在NameNode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts.exclude文件

```shell
[user_test@hadoop102 hadoop]$ pwd
/opt/module/hadoop-2.7.2/etc/hadoop
[user_test@hadoop102 hadoop]$ touch dfs.hosts.exclude
[user_test@hadoop102 hadoop]$ vi dfs.hosts.exclude
```

添加如下主机名称（要退役的节点）
hadoop105

2．在NameNode的hdfs-site.xml配置文件中增加dfs.hosts.exclude属性

```xml
<property>
	<name>dfs.hosts.exclude</name>
	<value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts.exclude</value>
</property>
```

3．刷新NameNode、刷新ResourceManager

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes
——>
Refresh nodes successful

[user_test@hadoop102 hadoop-2.7.2]$ yarn rmadmin -refreshNodes
——>
17/06/24 14:55:56 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033
```

4.	检查Web浏览器，退役节点的状态为decommission in progress（退役中），说明数据节点正在复制块到其他节点，如图:

![](7.png)

5.	等待退役节点状态为decommissioned（所有块已经复制完成），停止该节点及节点资源管理器。注意：如果副本数是3，服役的节点小于等于3，是不能退役成功的，需要修改副本数后才能退役，如图:

![](8.png)

可以看到hadoop105仍然还在页面上，但是里面Last contact是随着时间推移而增大的。这就表示上次连接时间与现在的间隔。

```shell
[user_test@hadoop105 hadoop-2.7.2]$ sbin/hadoop-daemon.sh stop datanode
——>
stopping datanode

[user_test@hadoop105 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager
——>
stopping nodemanager
```

6.	如果数据不均衡，可以用命令实现集群的再平衡

```shell
[user_test@hadoop102 hadoop-2.7.2]$ sbin/start-balancer.sh 
——>
starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out
Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved
```

**注意：不允许白名单和黑名单中同时出现同一个主机名称。**

### Datanode多目录配置

1.	DataNode也可以配置成多个目录，每个目录存储的数据不一样。即：数据不是副本

2．具体配置如下

hdfs-site.xml

```xml
<property>
	<name>dfs.datanode.data.dir</name>
	<value>file:///${hadoop.tmp.dir}/dfs/data1,file:///${hadoop.tmp.dir}/dfs/data2</value>
</property>
```

# 待续...

HDFS 2.X的一些新特性（集群间数据拷贝、小文件存档、回收站、快照管理）

> 这次DataNode相关直接一块推上去了，感觉这样会好点，吸取教训下次整块推。另外今天一个同学告诉我收到了某公司offer，也是自己比较中意的公司还是大数据开发相关的。嗯。真的不错，替他高兴，感觉他准备挺充分的，向他学习吧，希望一切顺利，到时候秋招我也能跟他一样~