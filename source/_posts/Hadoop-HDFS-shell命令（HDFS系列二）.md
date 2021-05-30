---
title: Hadoop(HDFS)shell命令（HDFS系列二）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-02 18:53:23
password:
summary: Hadoop之HDFS的Shell命令
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS 的 Shell 操作

1．基本语法

	bin/hadoop fs 具体命令   OR  bin/hdfs dfs 具体命令

dfs是fs的实现类。

2．命令大全

可通过如下命令进行查看

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs
```

3．常用命令实操

（1）启动Hadoop集群（方便后续的测试）

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
	[user_test@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
```

（2）-help：查询命令参数及功能

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -help rm
```

（3）-ls: 显示目录信息

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -ls /
```

递归查看

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -lsr /
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -ls -R /
```

（4）-mkdir：在HDFS上创建目录

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -mkdir -p /user/test/input
```

（5）-moveFromLocal：从本地剪切粘贴到HDFS

```shell
    [user_test@hadoop102 hadoop-2.7.2]$ touch test.txt
    [user_test@hadoop102 hadoop-2.7.2]$ hadoop fs  -moveFromLocal test.txt  /user/test/input
```

往 test.txt 文件中输入任意内容，等会测试追加

（6）-appendToFile：追加一个文件到已经存在的文件末尾

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ touch testAppend.txt
	[user_test@hadoop102 hadoop-2.7.2]$ vim testAppend.txt
```

输入任意内容

```shell
    [user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -appendToFile testAppend.txt /user/test/input/test.txt
```

（7）-cat：显示文件内容

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -cat /user/test/input/kongtest.txt
```

（8）-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs  -chmod  666 /user/test/input/test.txt
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs  -chown  user_test:user   /user/test/input/test.txt
```

（9）-copyFromLocal：从本地文件系统中拷贝文件到HDFS路径去

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -copyFromLocal README.txt /
```

（10）-copyToLocal：从HDFS拷贝到本地

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -copyToLocal /user/test/input/test.txt ./
```

（11）-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -cp /user/test/input/test.txt /user/test/input
```

（12）-mv：在HDFS目录中移动文件

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -mv /user/test/input/test.txt /
```

（13）-get：等同于copyToLocal，就是从HDFS下载文件到本地

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -get /user/test/input/test.txt ./
```

（14）-getmerge：合并下载多个文件，比如HDFS的目录 /user/user_test/test下有多个文件:log.1, log.2,log.3,...

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -getmerge /user/user_test/test/* ./merge.txt
```

（15）-put：等同于copyFromLocal

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -put ./merge.txt /user/test/input/
```

（16）-tail：显示一个文件的末尾

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -tail /user/test/input/test.txt
```

（17）-rm：删除文件或文件夹

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -rm /user/test/input/merge.txt
```

（18）-rmdir：删除空目录

```shell
    [user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -mkdir /test_null
    [user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -rmdir /test_null
```

（19）-du统计文件夹的大小信息

统计整个文件大小

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -du -s -h /user/test/input
```

	——> 2.7 K

分别列出大小

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -du -h /user/test/input
```

    ——>
    2.1.3 K  /user/test/input/README.txt
    15     /user/test/input/jinlian.txt
    1.4 K  /user/test/input/zaiyiqi.txt

（20）-setrep：设置HDFS中文件的副本数量

```shell
	[user_test@hadoop102 hadoop-2.7.2]$ hadoop fs -setrep 10 /user/test/input/test.txt
```

**这里设置的副本数只是记录在NameNode的元数据中，是否真的会有这么多副本，还得看DataNode的数量。因为目前只有3台设备，最多也就3个副本，只有节点数的增加到10台时，副本数才能达到10。**

# 待续...

接下来是HDFS客户端准备，应该是开发向重点了~

> 美好的周末又结束了，感觉下周将会是任务繁重的一周，一定要顺顺利利啊~
> 同时也要祝屁屁工作顺利哦~