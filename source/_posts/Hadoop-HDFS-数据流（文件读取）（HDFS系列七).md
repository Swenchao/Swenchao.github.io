---
title: Hadoop-HDFS-数据流（文件读取）（HDFS系列七）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-21 21:42:17
password:
summary: HDFS数据流（文件读取）
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS的数据流（面试重点）

### HDFS读数据流程

![](1.png)

其中 ss.avi 有三个副本，每个副本有两块。

1）客户端通过 Distributed FileSystem 向 NameNode 请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。

2）挑选一台 DataNode 服务器（就近原则），请求读取数据。若数据损坏，则选择一个副本来进行读取。

3）DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。

4）客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

# 待续

接下来就是 NameNode 和 SecondaryNameNode 相关（首先是工作机制然后是时间点设置再就是 NN 故障处理）

> 顺顺利利不挨批，收获多多~
> 屁屁明天也顺利