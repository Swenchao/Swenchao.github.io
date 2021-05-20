---
title: Hadoop-HDFS完整版笔记
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-03 20:44:19
password:
summary: 这是HDFS部分所有的笔记，其中图片都没有上传，因为之前单独部分都已经上传了。
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## 概述

感觉不是很重要，文件块大小那节要明白，其他稍微了解下就可以了~

### 产生背景及定义

![](d:\scwri\Desktop\1.png)

### 优缺点

![优点](d:\scwri\Desktop\2.png)

![缺点](d:\scwri\Desktop\3.png)

### 组成架构（之前概述讲过 ）

![](d:\scwri\Desktop\4.png)

![](d:\scwri\Desktop\5.png)

### 文件块大小

![](d:\scwri\Desktop\6.png)

**注：文件块不能设置太小也不能设置太大。因为如果设置太小，会增加寻址时间，程序会一直在寻找块开始的位置；若块设置太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。HDFS块的大小设置主要取决于磁盘的传输速率**

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

## HDFS客户端环境

### HDFS客户端环境准备

1．根据自己电脑的操作系统选择对应的编译后的hadoop jar包保存到非中文路径下（例如：D:\nonblank\hadoop-2.7.2）

2．配置HADOOP_HOME环境变量

![](1.png)

3. 配置Path环境变量

![](2.png)

4．创建一个Maven工程HdfsClientDemo

5．导入相应的依赖并添加日志

（1）导入依赖

```java
<dependencies>
	<dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.7.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>2.7.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-hdfs</artifactId>
        <version>2.7.2</version>
    </dependency>
    <dependency>
        <groupId>jdk.tools</groupId>
        <artifactId>jdk.tools</artifactId>
        <version>1.8</version>
        <scope>system</scope>
        <systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
    </dependency>
</dependencies>
```

（2）添加日志相关

在src/main/resources目录下，新建一个文件，命名为“log4j.properties”，填写如下内容：

    log4j.rootLogger=INFO, stdout
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
    log4j.appender.logfile=org.apache.log4j.FileAppender
    log4j.appender.logfile.File=target/spring.log
    log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
    log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n

6．创建包名：bigdata.hadoop.hdfs

7．创建HdfsClient类

```java
public class HdfsClient{	

public void testMkdirs() throws IOException, InterruptedException, URISyntaxException{
		
		// 1 获取文件系统
		Configuration configuration = new Configuration();
		
		// 配置在集群上运行(方式1)
		// configuration.set("fs.defaultFS", "hdfs://hadoop102:9000");
		// FileSystem fs = FileSystem.get(configuration);
	
		// 方式2(其中user_test为用户名)
		FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");
		
		// 2 创建目录
		fs.mkdirs(new Path("/user_test/test/sc"));
		
		// 3 关闭资源
		fs.close();
	}
}
```

8．执行程序

方式1运行时需要配置用户名称

![](3.png)

![](4.png)

客户端去操作HDFS时，是有一个用户身份的。默认情况下，HDFS客户端API会从JVM中获取一个参数来作为自己的用户身份：-DHADOOP_USER_NAME=user_test，user_test为用户名称。

### HDFS的API操作

#### HDFS文件上传（测试参数优先级）

1．编写源代码

```java
@Test
public void testCopyFromLocalFile() throws IOException, InterruptedException, URISyntaxException {

    // 1 获取文件系统
    Configuration configuration = new Configuration();
    configuration.set("dfs.replication", "2");
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");

    // 2 上传文件（第一个地址为本地的，第二个为hdfs上的地址）
    fs.copyFromLocalFile(new Path("test.txt"), new Path("/test.txt"));

    // 3 关闭资源
    fs.close();

    System.out.println("over");
}
```

2．将hdfs-site.xml拷贝到项目的根目录下（配置文件设置备份数量——这里是1）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
	<property>
		<name>dfs.replication</name>
        <value>1</value>
	</property>
</configuration>
```

**注：
参数优先级排序：（1）客户端代码中设置的值 >（2）ClassPath下的用户自定义配置文件 >（3）服务器的默认配置
**

#### HDFS文件下载

```java
@Test
public void testCopyToLocalFile() throws IOException, InterruptedException, URISyntaxException{

    // 1 获取文件系统
    Configuration configuration = new Configuration();
    FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");

    // 2 执行下载操作(第一个地址是hdfs地址，第二个为本地地址)
    //可选参数
    // boolean delSrc 指是否将原文件删除
    // Path src 指要下载的文件路径
    // Path dst 指将文件下载到的路径
    // boolean useRawLocalFileSystem 是否开启文件校验（若不开启就会产生一个crc的校验文件）
    fs.copyToLocalFile(false, new Path("/loadFile/test.txt"), new Path("test.txt"), true);

    // 3 关闭资源
    fs.close();
}
```

#### HDFS文件夹删除

```java
@Test
public void testDelete() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");
		
	// 2 执行删除(第一个是要删除的地址；若是删除的是个路径，则第二个必须为true，文件则无所谓)
	fs.delete(new Path("/test"), true);

	// 3 关闭资源
	fs.close();
}
```

#### HDFS文件名更改

```java
@Test
public void testRename() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test"); 
		
	// 2 修改文件名称(第一个为源文件名称，第二个为新文件名称)
	fs.rename(new Path("/loadFile/test.txt"), new Path("/loadFile/newTest.txt"));
		
	// 3 关闭资源
	fs.close();
}
```

#### HDFS文件详情查看
查看文件名称、权限、长度、块信息

```java
@Test
public void testListFiles() throws IOException, InterruptedException, URISyntaxException{

	// 1获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test"); 
		
	// 2 获取文件详情(第二个参数为递归查看)
	RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
	
	// 遍历迭代器
	while(listFiles.hasNext()){
		LocatedFileStatus status = listFiles.next();
			
		// 输出详情
		// 文件名称
		System.out.println(status.getPath().getName());
		// 长度
		System.out.println(status.getLen());
		// 权限
		System.out.println(status.getPermission());
		// 分组
		System.out.println(status.getGroup());
			
		// 获取存储的块信息
		BlockLocation[] blockLocations = status.getBlockLocations();
			
		for (BlockLocation blockLocation : blockLocations) {
				
			// 获取块存储的主机节点
			String[] hosts = blockLocation.getHosts();
				
			for (String host : hosts) {
				System.out.println(host);
			}
		}
			
		System.out.println("------------------------");
	}

	// 3 关闭资源
	fs.close();
}
```

#### HDFS文件和文件夹判断

```java
@Test
public void testListStatus() throws IOException, InterruptedException, URISyntaxException{
		
	// 1 获取文件配置信息
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");
		
	// 2 判断是文件还是文件夹
	FileStatus[] listStatus = fs.listStatus(new Path("/"));
		
	for (FileStatus fileStatus : listStatus) {
		
		// 如果是文件
		if (fileStatus.isFile()) {
			System.out.println("f:"+fileStatus.getPath().getName());
		}else {
			System.out.println("d:"+fileStatus.getPath().getName());
		}
	}
		
	// 3 关闭资源
	fs.close();
}
```

### HDFS的I/O流操作

上面api都是框架封装好的。现在可以尝试自己实现（我们可以采用IO流的方式实现数据的上传和下载）

#### HDFS文件上传

1．需求：把本地的test.txt文件上传到HDFS根目录

2．编写代码

```java
@Test
public void putFileToHDFS() throws IOException, InterruptedException, URISyntaxException {

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");

	// 2 创建输入流
	FileInputStream fis = new FileInputStream(new File("test.txt"));

	// 3 获取输出流
	FSDataOutputStream fos = fs.create(new Path("/test.txt"));

	// 4 流对拷
	IOUtils.copyBytes(fis, fos, configuration);

	// 5 关闭资源
	IOUtils.closeStream(fos);
	IOUtils.closeStream(fis);
    fs.close();
}
```

#### HDFS文件下载

1．需求：从HDFS上下载test.txt文件到本地

2．编写代码

```java
// 文件下载
@Test
public void getFileFromHDFS() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");
	
	// 输入流：写出的；输出流：写入的——在这个里面本地就是输出流，hdfs就是输出流
	
	// 2 获取输入流
	FSDataInputStream fis = fs.open(new Path("/loadFile/test.txt"));
		
	// 3 获取输出流
	FileOutputStream fos = new FileOutputStream(new File("test.txt"));
		
	// 4 流的对拷
	IOUtils.copyBytes(fis, fos, configuration);
		
	// 5 关闭资源
	IOUtils.closeStream(fos);
	IOUtils.closeStream(fis);
	fs.close();
}
```

#### 文件定位读取

1．需求：分块读取HDFS上的大文件，比如根目录下的/hadoop-2.7.2.tar.gz（首先手动将文件上传到hdfs——分成两块）

2．编写代码

```java
（1）下载第一块
@Test
public void readFileSeek1() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");
		
	// 2 获取输入流
	FSDataInputStream fis = fs.open(new Path("/hadoop-2.7.2.tar.gz"));
		
	// 3 创建输出流
	FileOutputStream fos = new FileOutputStream(new File("/hadoop-2.7.2.tar.gz.part1"));
		
	// 4 流的拷贝（若还用copyBytes来拷贝，则是拷贝的全部）
	// 相当与1k
	byte[] buf = new byte[1024];
	
	// 读取128M
	for(int i =0 ; i < 1024 * 128; i++){
		fis.read(buf);
		fos.write(buf);
	}
		
	// 5关闭资源
	IOUtils.closeStream(fis);
	IOUtils.closeStream(fos);
	fs.close();
}
```

```java
（2）下载第二块
@Test
public void readFileSeek2() throws IOException, InterruptedException, URISyntaxException{

	// 1 获取文件系统
	Configuration configuration = new Configuration();
	FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:9000"), configuration, "user_test");
		
	// 2 打开输入流
	FSDataInputStream fis = fs.open(new Path("/hadoop-2.7.2.tar.gz"));
		
	// 3 定位输入数据位置(128M位置处)
	fis.seek(1024*1024*128);
		
	// 4 创建输出流
	FileOutputStream fos = new FileOutputStream(new File("/hadoop-2.7.2.tar.gz.part2"));
		
	// 5 流的对拷
	IOUtils.copyBytes(fis, fos, configuration);
		
	// 6 关闭资源
	IOUtils.closeStream(fis);
	IOUtils.closeStream(fos);
	fs.close();
}
```

（3）合并文件

在Window命令窗口中进入到下载文件目录下，然后执行命令，进行数据合并

```shell
	type hadoop-2.7.2.tar.gz.part2 >> hadoop-2.7.2.tar.gz.part1
```

合并完成后，将hadoop-2.7.2.tar.gz.part1重新命名为hadoop-2.7.2.tar.gz。

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

### HDFS读数据流程

![](1.png)

其中 ss.avi 有三个副本，每个副本有两块。

1）客户端通过 Distributed FileSystem 向 NameNode 请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。

2）挑选一台 DataNode 服务器（就近原则），请求读取数据。若数据损坏，则选择一个副本来进行读取。

3）DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以 Packet 为单位来做校验）。

4）客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。

## NameNode和SecondaryNameNode（面试开发重点）

### NN和2NN工作机制

**思考：NameNode中的元数据是存储在哪里的？**

首先，我们做个假设，如果存储在NameNode节点的磁盘中，因为经常需要进行随机访问，还有响应客户请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的FsImage。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新FsImage，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦NameNode节点断电，就会产生数据丢失。因此，引入Edits文件(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到Edits中。这样，一旦NameNode节点断电，可以通过FsImage和Edits的合并，合成元数据。

但是，如果长时间添加数据到Edits中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行FsImage和Edits的合并，如果这个操作由NameNode节点完成，又会效率过低。因此，引入一个新的节点SecondaryNamenode，专门用于FsImage和Edits的合并。

NN和2NN工作机制，如下：

![](2.png)

1. 第一阶段：NameNode启动

（1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

（2）客户端对元数据进行增删改的请求。

（3）NameNode记录操作日志，更新滚动日志。

（4）NameNode在内存中对数据进行增删改。

**之所以先更新日志再进行操作，是因为如果先操作数据，未写日志突然断电，那么数据操作有可能丢失，无法还原。**

2. 第二阶段：Secondary NameNode工作

（1）Secondary NameNode 询问NameNode是否需要 CheckPoint。直接带回NameNode是否检查结果。
CheckPoint是一个检查点，检测是否需要合并日志文件和 fsimage 然后再序列化到 fsimge 中。

（2）若触发条件满足了，则 Secondary NameNode 请求执行CheckPoint。

（3）NameNode滚动正在编写的Edits日志，避免在合并的时候没法继续编写操作（001为原来的，然后新的日志先写到002中）。

（4）将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。

（5）Secondary NameNode 加载编辑日志和镜像文件到内存，并合并。

（6）生成新的镜像文件 fsimage.chkpoint。

（7）拷贝 fsimage.chkpoint 到 NameNode。

（8）NameNode 将 fsimage.chkpoint 重新命名成 fsimage（现在情况是002和新的镜像文件fsimage合起来就是当前内存中最新的内容）。

**NN和2NN工作机制详解**

Fsimage：NameNode内存中元数据序列化后形成的文件。

Edits：记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）。

NameNode 启动时，先滚动Edits并生成一个空的 edits.inprogress，然后加载Edits和Fsimage 到内存中，此时 NameNode 内存就持有最新的元数据信息。

Client 开始对NameNode 发送元数据的增删改的请求，这些请求的操作首先会被记录到edits.inprogress 中（查询元数据的操作不会被记录在Edits中，因为查询操作不会更改元数据信息），如果此时 NameNode 挂掉，重启后会从 Edits 中读取元数据的信息。然后，NameNode 会在内存中执行元数据的增删改的操作。

由于 Edits 中记录的操作会越来越多，Edits文件会越来越大，导致NameNode在启动加载Edits时会很慢，所以需要对 Edits 和 Fsimage 进行合并（所谓合并，就是将Edits和Fsimage加载到内存中，照着Edits中的操作一步步执行，最终形成新的Fsimage）。

SecondaryNameNode的作用就是帮助NameNode进行Edits和Fsimage的合并工作。
SecondaryNameNode首先会询问NameNode是否需要CheckPoint（触发CheckPoint需要满足两个条件中的任意一个，定时时间到和Edits中数据写满了）。直接带回NameNode是否检查结果。SecondaryNameNode执行CheckPoint操作，首先会让NameNode滚动Edits并生成一个空的edits.inprogress，滚动Edits的目的是给Edits打个标记，以后所有新的操作都写入edits.inprogress，其他未合并的Edits和Fsimage会拷贝到SecondaryNameNode的本地，然后将拷贝的Edits和Fsimage加载到内存中进行合并，生成fsimage.chkpoint，然后将fsimage.chkpoint拷贝给NameNode，重命名为Fsimage后替换掉原来的Fsimage。

NameNode在启动时就只需要加载之前未合并的Edits和Fsimage即可，因为合并过的Edits中的元数据信息已经被记录在Fsimage中。

### Fsimage和Edits解析

1. 概念

![](3.png)

2. oiv查看Fsimage文件

（1）查看oiv和oev命令

```shell
[user_test@hadoop102 current]$ hdfs
——>
...
oiv		apply the offline fsimage viewer to an fsimage
oev		pply the offline edits viewer to an edits file
...
```

（2）基本语法

```shell
hdfs oiv -p （要将 fsimage 转化成的格式） -i （镜像文件） -o （转换后文件输出路径）
```

（3）案例实操（其中文件名称可能有区别）

```shell
[user_test@hadoop102 current]$ pwd
——>
/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current

[user_test@hadoop102 current]$ hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-2.7.2/fsimage.xml

[user_test@hadoop102 current]$ cat /opt/module/hadoop-2.7.2/fsimage.xml
```

将显示的xml文件内容拷贝到idea中创建的xml文件中，并格式化。部分显示结果如下（代码信息会有所区别）。

```xml
<inode>
	<id>16386</id>
	<type>DIRECTORY</type>
	<name>user</name>
	<mtime>1512722284477</mtime>
	<permission>user_test:supergroup:rwxr-xr-x</permission>
	<nsquota>-1</nsquota>
	<dsquota>-1</dsquota>
</inode>
<inode>
	<id>16387</id>
	<type>DIRECTORY</type>
	<name>user_test</name>
	<mtime>1512790549080</mtime>
	<permission>user_test:supergroup:rwxr-xr-x</permission>
	<nsquota>-1</nsquota>
	<dsquota>-1</dsquota>
</inode>
<inode>
	<id>16389</id>
	<type>FILE</type>
	<name>wc.input</name>
	<replication>3</replication>
	<mtime>1512722322219</mtime>
	<atime>1512722321610</atime>
	<perferredBlockSize>134217728</perferredBlockSize>
	<permission>user_test:supergroup:rw-r--r--</permission>
	<blocks>
		<block>
			<id>1073741825</id>
			<genstamp>1001</genstamp>
			<numBytes>59</numBytes>
		</block>
	</blocks>
</inode >
```

**思考：可以看出，Fsimage中没有记录块所对应DataNode，为什么？**

在集群启动后，要求DataNode上报数据块信息，并间隔一段时间后再次上报。（datanode运行机制会详细说）

3. oev查看Edits文件

（1）基本语法

```shell
hdfs oev -p （要将 fsimage 转化成的格式） -i （镜像文件） -o （转换后文件输出路径）
```

（2）案例实操

```shell
[user_test@hadoop102 current]$ hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-2.7.2/edits.xml

[user_test@hadoop102 current]$ cat /opt/module/hadoop-2.7.2/edits.xml
```
将显示的xml文件内容拷贝到 idea 中创建的xml文件中，并格式化。显示结果如下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<EDITS>
	<EDITS_VERSION>-63</EDITS_VERSION>
	<RECORD>
		<OPCODE>OP_START_LOG_SEGMENT</OPCODE>
		<DATA>
			<TXID>129</TXID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ADD</OPCODE>
		<DATA>
			<TXID>130</TXID>
			<LENGTH>0</LENGTH>
			<INODEID>16407</INODEID>
			<PATH>/hello7.txt</PATH>
			<REPLICATION>2</REPLICATION>
			<MTIME>1512943607866</MTIME>
			<ATIME>1512943607866</ATIME>
			<BLOCKSIZE>134217728</BLOCKSIZE>
			<CLIENT_NAME>DFSClient_NONMAPREDUCE_-1544295051_1</CLIENT_NAME>
			<CLIENT_MACHINE>192.168.1.5</CLIENT_MACHINE>
			<OVERWRITE>true</OVERWRITE>
			<PERMISSION_STATUS>
				<USERNAME>user_test</USERNAME>
				<GROUPNAME>supergroup</GROUPNAME>
				<MODE>420</MODE>
			</PERMISSION_STATUS>
			<RPC_CLIENTID>908eafd4-9aec-4288-96f1-e8011d181561</RPC_CLIENTID>
			<RPC_CALLID>0</RPC_CALLID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ALLOCATE_BLOCK_ID</OPCODE>
		<DATA>
			<TXID>131</TXID>
			<BLOCK_ID>1073741839</BLOCK_ID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_SET_GENSTAMP_V2</OPCODE>
		<DATA>
			<TXID>132</TXID>
			<GENSTAMPV2>1016</GENSTAMPV2>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_ADD_BLOCK</OPCODE>
		<DATA>
			<TXID>133</TXID>
			<PATH>/hello7.txt</PATH>
			<BLOCK>
				<BLOCK_ID>1073741839</BLOCK_ID>
				<NUM_BYTES>0</NUM_BYTES>
				<GENSTAMP>1016</GENSTAMP>
			</BLOCK>
			<RPC_CLIENTID></RPC_CLIENTID>
			<RPC_CALLID>-2</RPC_CALLID>
		</DATA>
	</RECORD>
	<RECORD>
		<OPCODE>OP_CLOSE</OPCODE>
		<DATA>
			<TXID>134</TXID>
			<LENGTH>0</LENGTH>
			<INODEID>0</INODEID>
			<PATH>/hello7.txt</PATH>
			<REPLICATION>2</REPLICATION>
			<MTIME>1512943608761</MTIME>
			<ATIME>1512943607866</ATIME>
			<BLOCKSIZE>134217728</BLOCKSIZE>
			<CLIENT_NAME></CLIENT_NAME>
			<CLIENT_MACHINE></CLIENT_MACHINE>
			<OVERWRITE>false</OVERWRITE>
			<BLOCK>
				<BLOCK_ID>1073741839</BLOCK_ID>
				<NUM_BYTES>25</NUM_BYTES>
				<GENSTAMP>1016</GENSTAMP>
			</BLOCK>
			<PERMISSION_STATUS>
				<USERNAME>user_test</USERNAME>
				<GROUPNAME>supergroup</GROUPNAME>
				<MODE>420</MODE>
			</PERMISSION_STATUS>
		</DATA>
	</RECORD>
</EDITS >
```

**说明：**

    OP_START_LOG_SEGMENT：日志开始
    OP_MKDIR：创建路径
    OP_ADD：上传文件
    OP_ALLOCATE_BLOCK_ID：分配块 id
    OP_SET_GENSTAMP_V2：创建时间戳
    OP_ADD_BLOCK：添加块信息
    OP_CLOSE：操作结束
    OP_RENAME_OLD：重命名原来复制的文件

**思考：NameNode如何确定下次开机启动的时候合并哪些Edits？**

根据 seen_txid 文件中数据来合并，其中记录了最新的。

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

4. 如果 SecondaryNameNode 不和 NameNode 在一个主机节点上，需要将 SecondaryNameNode 存储数据的目录拷贝到 NameNode 存储数据的平级目录，并删除 in_use.lock 文件

```shell
[user_test@hadoop102 dfs]$ scp -r user_test@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary ./

[user_test@hadoop102 namesecondary]$ rm -rf in_use.lock

[user_test@hadoop102 dfs]$ pwd
/opt/module/hadoop-2.7.2/data/tmp/dfs

[user_test@hadoop102 dfs]$ ls
data  name  namesecondary
```

5. 导入检查点数据（等待一会ctrl+c结束掉）

```shell
[user_test@hadoop102 hadoop-2.7.2]$ bin/hdfs namenode -importCheckpoint
```

6. 启动NameNode

```shell
[user_test@hadoop102 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode
```

### 集群安全模式

1. 概述

![](4.png)

2. 基本语法

集群处于安全模式，不能执行重要操作（写操作）。集群启动完成后，自动退出安全模式。

```shell
	（1）bin/hdfs dfsadmin -safemode get		（查看安全模式状态）
	（2）bin/hdfs dfsadmin -safemode enter  	（进入安全模式状态）
	（3）bin/hdfs dfsadmin -safemode leave	（离开安全模式状态）
	（4）bin/hdfs dfsadmin -safemode wait		（等待安全模式状态）
```

3. 案例

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

（a）再观察上一个窗口

Safe mode is OFF

（b）HDFS集群上已经有上传的数据了。

### NameNode多目录配置

1. NameNode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性

2. 具体配置如下

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

## DataNode（面试开发重点）

### DataNode工作机制

![](1.png)

一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

1）DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。

2）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令（如复制块数据到另一台机器，或删除某个数据块）。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。

3）集群运行中可以安全加入和退出一些机器。

### 数据完整性

**思考：如果电脑磁盘里面存储的数据是控制高铁信号灯的红灯信号（1）和绿灯信号（0），但是存储该数据的磁盘坏了，一直显示是绿灯，是否很危险？同理DataNode节点上的数据损坏了，却没有发现，是否也很危险，那么如何解决呢？**

如下是DataNode节点保证数据完整性的方法。

1）当DataNode读取Block的时候，它会计算CheckSum。

2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。

3）Client读取其他DataNode上的Block。

4）DataNode在其文件创建后周期验证CheckSum，如图：

![](2.png)

### 掉线时限参数设置

![](4.png)

需要注意的是hdfs-site.xml 配置文件中的 heartbeat.recheck.interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒。

```xml
<property>
    <name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>300000</value>
</property>
<property>
    <name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>
```

### 服役新数据节点

0. 需求
随着公司业务的增长，数据量越来越大，原有的数据节点的容量已经不能满足存储数据的需求，需要在原有集群基础上动态添加新的数据节点。

1.	环境准备

（1）在hadoop104主机上再克隆一台hadoop105主机

（2）修改IP地址和主机名称

（3）删除原来HDFS文件系统留存的文件（/opt/module/hadoop-2.7.2/data和log）

（4）source一下配置文件

```shell
[user_test@hadoop105 hadoop-2.7.2]$ source /etc/profile
```

2.	服役新节点具体步骤

（1）直接启动DataNode，即可关联到集群

```shell
[user_test@hadoop105 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start datanode

[user_test@hadoop105 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager
```

![](5.png)

（2）在hadoop105上上传文件

```shell
[user_test@hadoop105 hadoop-2.7.2]$ hadoop fs -put /opt/module/hadoop-2.7.2/LICENSE.txt /
```

（3）如果数据不均衡，可以用命令实现集群的再平衡

```shell
[user_test@hadoop102 sbin]$ ./start-balancer.sh
——>
starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out
Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved
```

### 退役旧数据节点

#### 添加白名单

添加到白名单的主机节点，都允许访问NameNode，不在白名单的主机节点，都会被退出。

配置白名单的具体步骤如下：

（1）在NameNode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts文件，添加以下内容

```shell
[user_test@hadoop102 hadoop]$ pwd
——>
/opt/module/hadoop-2.7.2/etc/hadoop

[user_test@hadoop102 hadoop]$ touch dfs.hosts

[user_test@hadoop102 hadoop]$ vi dfs.hosts
```

添加如下主机名称（不添加hadoop105）
hadoop102
hadoop103
hadoop104

（2）在NameNode的hdfs-site.xml配置文件中增加dfs.hosts属性

```xml
<property>
	<name>dfs.hosts</name>
	<value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts</value>
</property>
```

（3）配置文件分发

```shell
[user_test@hadoop102 hadoop]$ xsync hdfs-site.xml
```

（4）刷新NameNode

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes
——>
Refresh nodes successful
```

（5）更新ResourceManager节点

```shell
[user_test@hadoop103 hadoop-2.7.2]$ yarn rmadmin -refreshNodes
——>
17/06/24 14:17:11 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033
```

（6）在web浏览器上查看

![](6.png)

4.	如果数据不均衡，可以用命令实现集群的再平衡

```shell
[user_test@hadoop102 sbin]$ ./start-balancer.sh
——>
starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out
Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved
```

#### 黑名单退役

**在做以下操作时，要注意退回到上一节操作之前状态**

在黑名单上面的主机都会被强制退出。

1.在NameNode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts.exclude文件

```shell
[user_test@hadoop102 hadoop]$ pwd
/opt/module/hadoop-2.7.2/etc/hadoop
[user_test@hadoop102 hadoop]$ touch dfs.hosts.exclude
[user_test@hadoop102 hadoop]$ vi dfs.hosts.exclude
```

添加如下主机名称（要退役的节点）
hadoop105

2．在NameNode的hdfs-site.xml配置文件中增加dfs.hosts.exclude属性

```xml
<property>
	<name>dfs.hosts.exclude</name>
	<value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts.exclude</value>
</property>
```

3．刷新NameNode、刷新ResourceManager

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes
——>
Refresh nodes successful

[user_test@hadoop102 hadoop-2.7.2]$ yarn rmadmin -refreshNodes
——>
17/06/24 14:55:56 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033
```

4.	检查Web浏览器，退役节点的状态为decommission in progress（退役中），说明数据节点正在复制块到其他节点，如图:

![](7.png)

5.	等待退役节点状态为decommissioned（所有块已经复制完成），停止该节点及节点资源管理器。注意：如果副本数是3，服役的节点小于等于3，是不能退役成功的，需要修改副本数后才能退役，如图:

![](8.png)

可以看到hadoop105仍然还在页面上，但是里面Last contact是随着时间推移而增大的。这就表示上次连接时间与现在的间隔。

```shell
[user_test@hadoop105 hadoop-2.7.2]$ sbin/hadoop-daemon.sh stop datanode
——>
stopping datanode

[user_test@hadoop105 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager
——>
stopping nodemanager
```

6.	如果数据不均衡，可以用命令实现集群的再平衡

```shell
[user_test@hadoop102 hadoop-2.7.2]$ sbin/start-balancer.sh 
——>
starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out
Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved
```

**注意：不允许白名单和黑名单中同时出现同一个主机名称。**

### Datanode多目录配置

1.	DataNode也可以配置成多个目录，每个目录存储的数据不一样。即：数据不是副本

2．具体配置如下

hdfs-site.xml

```xml
<property>
	<name>dfs.datanode.data.dir</name>
	<value>file:///${hadoop.tmp.dir}/dfs/data1,file:///${hadoop.tmp.dir}/dfs/data2</value>
</property>
```

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
