---
title: Hadoop(HDFS)概述（HDFS系列一）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-02 18:52:54
password:
summary: Hadoop——HDFS概述
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## 概述

感觉不是很重要，文件块大小那节要明白，其他稍微了解下就可以了~

### 产生背景及定义

![](1.png)

### 优缺点

![优点](2.png)

![缺点](3.png)

### 组成架构（之前概述讲过 ）

![](4.png)

![](5.png)

### 文件块大小

![](6.png)

**注：文件块不能设置太小也不能设置太大。因为如果设置太小，会增加寻址时间，程序会一直在寻找块开始的位置；若块设置太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。HDFS块的大小设置主要取决于磁盘的传输速率**

# 待续...

接下来就是HDFS的一些命令操作了~