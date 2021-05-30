---
title: Hadoop-HDFS-数据流（文件写入、网络拓扑）（HDFS系列六）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-20 23:01:45
password:
summary: HDFS数据流（文件写入、网络拓扑节点距离计算）
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS的数据流（面试重点）

### HDFS写数据流程

#### 剖析文件写入

HDFS写数据流程，如下

![](1.png)

1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。

2）NameNode返回是否可以上传。

3）客户端请求第一个 Block上传到哪几个DataNode服务器上。

4）NameNode返回3个DataNode节点，分别为dn1、dn2、dn3（备份）。

5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。

6）dn1、dn2、dn3逐级应答客户端。

7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3（这个过程是通过内存缓存来传的，因为速度快）；dn1每传一个packet会放入一个应答队列等待应答。

8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。

#### 网络拓扑-节点距离计算

在HDFS写数据的过程中，NameNode会选择距离待上传数据最近距离的DataNode接收数据。

**节点距离：两个节点到达最近的共同祖先的距离总和。**

![](2.png)

#### 机架感知（副本存储节点选择）

1. 官方ip地址

机架感知说明

http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Data_Replication

For the common case, when the replication factor is three, HDFS’s placement policy is to put one replica on one node in the local rack, another on a different node in the local rack, and the last on a different node in a different rack.

2. Hadoop2.7.2副本节点选择

![](3.png)

# 待续...

接下来就是HDFS读数据流程了

> 明天顺顺利利
> 屁屁团建开心，工作顺利~