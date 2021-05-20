---
title: Hadoop-HDFS-时间点设置以及NN故障处理
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-24 21:59:29
password:
summary: HDFS时间点（CheckPoint）以及NN故障处理
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## NameNode和SecondaryNameNode（面试开发重点）

### CheckPoint时间设置

（1）通常情况下，SecondaryNameNodec每隔一小时执行一次(3600秒)。

配置文件：hdfs-default.xml

```xml
<property>
	<name>dfs.namenode.checkpoint.period</name>
	<value>3600</value>
</property>
```

（2）一分钟检查一次操作次数，当操作次数达到1百万时，SecondaryNameNode 执行一次。

```xml
<property>
	<name>dfs.namenode.checkpoint.txns</name>
	<value>1000000</value>
	<description>操作动作次数</description>
</property>

<property>
	<name>dfs.namenode.checkpoint.check.period</name>
	<value>60</value>
	<description> 1分钟检查一次操作次数</description>
</property >
```

### NameNode故障处理

NameNode故障后，可以采用如下两种方法恢复数据。

#### 方法一：将 SecondaryNameNode 中数据拷贝到 NameNode 中进行数据备份或恢复；

1. kill -9 NameNode进程号

2. 删除NameNode存储的数据（/opt/module/hadoop-2.7.2/data/tmp/dfs/name）

```shell
[user_test@hadoop102 hadoop-2.7.2]$ rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*
```

3. 拷贝SecondaryNameNode中数据到原NameNode存储数据目录

```shell
[user_test@hadoop102 dfs]$ scp -r user_test@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/* ./name/
```

4. 重新启动NameNode

```shell
[atguigu@hadoop102 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
```

#### 方法二：使用 -importCheckpoint 选项启动 NameNode 守护进程，从而将 SecondaryNameNode 中数据拷贝到 NameNode 目录中。

1.	修改hdfs-site.xml中的（因为原来的3600秒时间太长了，所以改成120）

```xml
<property>
	<name>dfs.namenode.checkpoint.period</name>
	<value>120</value>
</property>

<property>
	<name>dfs.namenode.name.dir</name>
	<value>/opt/module/hadoop-2.7.2/data/tmp/dfs/name</value>
</property>
```

2.  kill -9 NameNode 进程

3.	删除NameNode存储的数据（/opt/module/hadoop-2.7.2/data/tmp/dfs/name）

```shell
[user_test@hadoop102 hadoop-2.7.2]$ rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*
```

4.	如果 SecondaryNameNode 不和 NameNode 在一个主机节点上，需要将 SecondaryNameNode 存储数据的目录拷贝到 NameNode 存储数据的平级目录，并删除 in_use.lock 文件

```shell
[user_test@hadoop102 dfs]$ scp -r user_test@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary ./

[user_test@hadoop102 namesecondary]$ rm -rf in_use.lock

[user_test@hadoop102 dfs]$ pwd
/opt/module/hadoop-2.7.2/data/tmp/dfs

[user_test@hadoop102 dfs]$ ls
data  name  namesecondary
```

5.	导入检查点数据（等待一会ctrl+c结束掉）

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hdfs namenode -importCheckpoint
```

6.	启动NameNode

```shell
[user_test@hadoop102 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
```

# 待续...

集群安全模式

> 顺顺利利，有自己满意结果~