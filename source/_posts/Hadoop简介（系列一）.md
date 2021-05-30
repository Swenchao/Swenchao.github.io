---
title: Hadoop简介（系列一）
top: true
cover: true
toc: true
mathjax: true
date: 2020-07-29 18:37:48
password:
summary: 开始学习整理Hadoop——Hadoop简介
tags:
- Hadoop
categories:
- 大数据
---

之前学过一段时间，好久没用，已经忘得干干净净了。现在只能重新开始，整理学习了~

# Hadoop简介

## HDFS架构概述

1. NameNode（nn）：存储文件元数据，以及每个文件的块列表和所在的DataNode等（方便找数据的索引）
2. DataNode（dn）：在本地文件系统存储文件块数据，以及数据的校验和（存数据）
3. Secondary NameNode（2nn）：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照（并不完全是NameNode的备份，当nn挂了，它只能辅助恢复，并不能完全替代，就像是主治医生和递手术刀的）

## YARN架构概述

调度CPU算力和内存资源

1. ResourceManager（RM）（集群中只有一个，管理整个项目所有调度）

（1）处理客户端请求（所有请求都会先发给它——作业提交）
（2）监控NodeManager
（3）启动或监控ApplicationMaster
（4）资源分配与调度

2. NodeManager（NM）

（1）管理单个节点上的资源
（2）处理来自RM的命令
（3）处理来自ApplicationMaster的命令

3. ApplicationMaster（AM）（项目临时负责任务跟进，非常住进程，一个job就是一个AM）

（1）负责数据切分
（2）为应用程序申请资源并分配给内部任务
（3）任务监控与容错

4. Container（非常主进程）

YARN中资源抽象，封装了某个节点上的多维度资源，如：内存、CPU、磁盘、网络

## MapReduce架构概述

MapReduce将计算过程分为两个阶段：Map（分）和Reduce（汇总）

![](1.jpg)

## 大数据生态体系

![](2.jpg)

HBase：类似于大表格
Hive、Mahout...：计算引擎（MapReduce）的包装（类似于mabatis是mysql的包装）
Kafka是在线计算（流式计算，无穷尽）：评价标准是实时处理速度（口径）；其他为离线计算

	Storm是纯流式计算——来一点处理一点（口径小）
	Spark不是纯流式处理，先存一部分，然后处理（Flink与其类似）

Zookeeper：协调各个框架之间关系

# 接下来

接下来要做的是，hadoop基本环境搭建（虚拟机、jdk以及hadoop安装）

>一切都要顺顺利利呀~