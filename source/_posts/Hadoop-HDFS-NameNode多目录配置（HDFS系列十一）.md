---
title: Hadoop-HDFS-NameNode多目录配置（HDFS系列十一）
top: false
cover: true
toc: true
mathjax: true
date: 2020-08-26 19:41:32
password:
summary: HDFS NameNode多目录配置
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## NameNode和SecondaryNameNode（面试开发重点）

### NameNode多目录配置

1.	NameNode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性

2.	具体配置如下

（1）在hdfs-site.xml文件中增加如下内容

```xml
<property>
	<name>dfs.namenode.name.dir</name>
	<value>file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2</value>
</property>
```

（2）停止集群，删除data和logs中所有数据。

```shell
[user_test@hadoop102 hadoop-2.7.2]$ rm -rf data/ logs/
[user_test@hadoop103 hadoop-2.7.2]$ rm -rf data/ logs/
[user_test@hadoop104 hadoop-2.7.2]$ rm -rf data/ logs/
```

（3）格式化集群并启动。

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hdfs namenode –format
[user_test@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
```

（4）查看结果

```shell
[user_test@hadoop102 dfs]$ ll
——>
总用量 12
drwx------. 3 user_test user_test 4096 12月 11 08:03 data
drwxrwxr-x. 3 user_test user_test 4096 12月 11 08:03 name1
drwxrwxr-x. 3 user_test user_test 4096 12月 11 08:03 name2
```

# 待续...

DataNode 相关（工作机制——>数据完整性——>掉线时限参数设置——>服役新数据节点——>退役旧数据节点）