---
title: Spark概述（Spark系列一）
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-21 21:30:11
password:
summary: Spark概述
tags:
- Spark
categories:
- 大数据
---

要开始学习Spark了，准备开启新的一系列了，一切顺利呀~

# Spark

## Spark概述

### 什么是 Spark

Spark 是一个快速(基于内存), 通用, 可扩展的集群计算引擎

并且 Spark 目前已经成为 Apache 最活跃的开源项目, 有超过 1000 个活跃的贡献者.

**历史**

2009 年，Spark 诞生于 UC Berkeley(加州大学伯克利分校, CAL) 的 AMP 实验室, 项目采用 Scala 编程语言编写。

2010 年, Spark 正式对外开源

2013 年 6 月, 进入 Apache 孵化器

2014 年, 成为 Apache 的顶级项目.

目前最新的版本是 2.4.0

参考: http://spark.apache.org/history.html

### Spark 内置模块介绍

![](1.png)

独立调度器：Spark自己的
YARN：HAdoop的

#### 集群管理器(Cluster Manager)

Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。

为了实现这样的要求，同时获得最大灵活性，Spark 支持在各种集群管理器(Cluster Manager)上运行，目前 Spark 支持 3 种集群管理器:

1. Hadoop YARN(在国内使用最广泛)

2. Apache Mesos(国内使用较少, 国外使用较多)

3. Standalone(Spark 自带的资源调度器, 需要在集群中的每台节点上配置 Spark)

#### SparkCore

实现了 Spark 的基本功能，包含任务调度、内存管理、错误恢复、与存储系统交互等模块。SparkCore 中还包含了对弹性分布式数据集(Resilient Distributed DataSet，简称RDD)的API定义。

#### Spark SQL

是 Spark 用来操作结构化数据的程序包。通过SparkSql，我们可以使用 SQL或者Apache Hive 版本的 SQL 方言(HQL)来查询数据。Spark SQL 支持多种数据源，比如 Hive 表、Parquet 以及 JSON 等。

#### Spark Streaming

是 Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API，并且与 Spark Core 中的 RDD API 高度对应。

#### Spark MLlib

提供常见的机器学习 (ML) 功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。

Spark 得到了众多大数据公司的支持，这些公司包括 Hortonworks、IBM、Intel、Cloudera、MapR、Pivotal、百度、阿里、腾讯、京东、携程、优酷土豆。

当前百度的 Spark 已应用于大搜索、直达号、百度大数据等业务；阿里利用 GraphX 构建了大规模的图计算和图挖掘系统，实现了很多生产系统的推荐算法；腾讯Spark集群达到 8000 台的规模，是当前已知的世界上最大的 Spark 集群。

### Spark 特点

Spark特点

1）快：与Hadoop的MapReduce相比，Spark基于内存的运算要快100倍以上，基于硬盘的运算也要快10倍以上。Spark实现了高效的DAG执行引擎，可以通过基于内存来高效处理数据流。计算的中间结果是存在于内存中的。

2）易用：Spark支持Java、Python和Scala的API，还支持超过80种高级算法，使用户可以快速构建不同的应用。而且Spark支持交互式的Python和Scala的Shell，可以非常方便地在这些Shell中使用Spark集群来验证解决问题的方法。

3）通用：Spark提供了统一的解决方案。Spark可以用于批处理、交互式查询（Spark SQL）、实时流处理（Spark Streaming）、机器学习（Spark MLlib）和图计算（GraphX）。这些不同类型的处理都可以在同一个应用中无缝使用。减少了开发和维护的人力成本和部署平台的物力成本。

4）兼容性：Spark可以非常方便地与其他的开源产品进行融合。比如，Spark可以使用Hadoop的YARN和Apache Mesos作为它的资源管理和调度器，并且可以处理所有Hadoop支持的数据，包括HDFS、HBase等。这对于已经部署Hadoop集群的用户特别重要，因为不需要做任何数据迁移就可以使用Spark的强大处理能力。

# 待续...

Spark运行模式