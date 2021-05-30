---
title: Hadoop-HDFS-HDFS2.X新特性（系列十三）
top: false
cover: true
toc: true
mathjax: true
date: 2020-09-03 19:37:12
password:
summary: HDFS2.X新特性
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS 2.X新特性

### 集群间数据拷贝

1．scp实现两个远程主机之间的文件复制

```shell
scp -r hello.txt root@hadoop103:/user/atguigu/hello.txt		// 推 push

scp -r root@hadoop103:/user/atguigu/hello.txt  hello.txt		// 拉 pull

scp -r root@hadoop103:/user/atguigu/hello.txt root@hadoop104:/user/atguigu   //是通过本地主机中转实现两个远程主机的文件复制；如果在两个远程主机之间ssh没有配置的情况下可以使用该方式。
```

2．采用distcp命令实现两个Hadoop集群之间的递归数据复制

```shell
[user_test@hadoop102 hadoop-2.7.2]$  bin/hadoop distcp
hdfs://haoop102:9000/user/atguigu/hello.txt hdfs://hadoop103:9000/user/atguigu/hello.txt
```

其中第一个地址为文件源地址，第二个为文件目的地址

### 小文件存档

![](1.png)

HAR文件就是HDFS归档文件，只是文件形式是这个。

3．案例实操

（1）需要启动YARN进程

```shell
[user_test@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```

（2）归档文件

把/user/test/input目录里面的所有文件归档成一个叫input.har的归档文件，并把归档后文件存储到/user/test/output路径下。（以上几个路径都得自己新建来测试）

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hadoop archive -archiveName input.har –p  /user/atguigu/input   /user/atguigu/output
```

其中 input.har 为要生成的har文件；-p后面第一个地址是源地址，第二个地址为将要生成存储的地址

（3）查看归档

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -lsr /user/atguigu/output/input.har
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -lsr har:///user/atguigu/output/input.har
```

第二个就会看到其中内容

（4）解归档文件

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -cp har:/// user/test/output/input.har/*    /user/test
```

### 回收站

开启回收站功能，可以将删除的文件在不超时的情况下，恢复原数据，起到防止误删除、备份等作用。

1．回收站参数设置及工作机制

![](2.png)

其中两个参数单位都是分钟

2．启用回收站

修改core-site.xml，配置垃圾回收时间为1分钟。

```shell
<property>
	<name>fs.trash.interval</name>
	<value>1</value>
</property>
```

3．查看回收站

回收站在集群中的路径：/user/atguigu/.Trash/….

这样会说权限不够，因此需要下一步来配置权限

4．修改访问垃圾回收站用户名称

进入垃圾回收站用户名称，默认是dr.who，修改为user_test用户

[core-site.xml]

```xml
<property>
  <name>hadoop.http.staticuser.user</name>
  <value>user_test</value>
</property>
```

5.	通过程序删除的文件不会经过回收站，需要调用moveToTrash()才进入回收站

Trash trash = New Trash(conf);
trash.moveToTrash(path);

5.	恢复回收站数据

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -mv
/user/user_test/.Trash/Current/user/test/input    /user/test/input
```

6.	清空回收站

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -expunge
```

并不是跟windows一样全部清空，二十重新打一个包，放起来。

### 快照管理

![](3.png)

禁用快照功能的时候，要先删除已有快照

2．案例实操

（1）开启/禁用指定目录的快照功能

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -allowSnapshot /user/test/input
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -disallowSnapshot /user/test/input
```

（2）对目录创建快照

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfs -createSnapshot /user/test/input
——>
Created snapshot /user/test/input/.snapshot/s20200903-165659.207

[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfs -lsr /user/test/input/.snapshot/
```

快照放在一个隐藏文件之中

![](4.png)

（3）指定名称创建快照

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfs -createSnapshot /user/atguigu/input  scSnapshot
```

（4）重命名快照

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfs -renameSnapshot /user/atguigu/input/  scSnapshot sc20200903
```

（5）列出当前用户所有可快照目录

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs lsSnapshottableDir
```

（6）比较两个快照目录的不同之处

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs snapshotDiff
 /user/test/input/  .  .snapshot/sc20200903	
——>
Difference between current directory and snapshot s20200903-165659.207 under directory /user/test/input:
M       .
-       ./README.txt
```

```
- ./README.txt 
```
这句就表示了，现有快照比现在文件少了一个 readme.txt 文件

（7）恢复快照

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfs -cp
/user/atguigu/input/.snapshot/s20170708-134303.027 /user
```

# 待续...

HDFS部分暂时要告一段落了，接下来是MapReduce了，还有好长的路要走~
稍后会把HDFS一个合订版放上来。

>一切顺利，有所收获~