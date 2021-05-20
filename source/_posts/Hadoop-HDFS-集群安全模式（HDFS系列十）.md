---
title: Hadoop-HDFS-集群安全模式（HDFS系列十）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-25 19:13:49
password:
summary: HDFS 集群安全模式
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## NameNode和SecondaryNameNode（面试开发重点）

### 集群安全模式

1.	概述

![](4.png)

2.	基本语法

集群处于安全模式，不能执行重要操作（写操作）。集群启动完成后，自动退出安全模式。

```shell
	（1）bin/hdfs dfsadmin -safemode get		（查看安全模式状态）
	（2）bin/hdfs dfsadmin -safemode enter  	（进入安全模式状态）
	（3）bin/hdfs dfsadmin -safemode leave	（离开安全模式状态）
	（4）bin/hdfs dfsadmin -safemode wait		（等待安全模式状态）
```

3.	案例

模拟等待安全模式（在安全模式时间内，来了任务，需要让其等待安全模式结束，然后立即执行。比如：银行对账时，在凌晨进行，在对账这个过程中，是不允许进行任何交易的，因此需要等待对账结束，然后）

（1）查看当前模式

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -safemode get
——>
Safe mode is OFF
```
（2）先进入安全模式

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hdfs dfsadmin -safemode enter
——>
Safe mode is ON
```

上传文件进行测试

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hdfs dfs -put sc.txt /loadFile/
——>
put: Cannot create file/loadFile/sc.txt._COPYING_. Name node is in safe mode.
```

（3）创建并执行下面的脚本

在/opt/module/hadoop-2.7.2路径上，创建一个脚本safemode.sh

```shell
[user_test@hadoop102 hadoop-2.7.2]$ touch safemode.sh
[user_test@hadoop102 hadoop-2.7.2]$ vim safemode.sh
```

输入内容

```xml
#!/bin/bash
hdfs dfsadmin -safemode wait
hdfs dfs -put /opt/module/hadoop-2.7.2/README.txt /
```

```shell
[user_test@hadoop102 hadoop-2.7.2]$ chmod 777 safemode.sh

[user_test@hadoop102 hadoop-2.7.2]$ bash safemode.sh 
```

（4）再打开一个窗口，执行

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hdfs dfsadmin -safemode leave
```

（5）观察

a）再观察上一个窗口

Safe mode is OFF

b）HDFS集群上已经有上传的数据了。

# 待续...

NameNode多目录配置
