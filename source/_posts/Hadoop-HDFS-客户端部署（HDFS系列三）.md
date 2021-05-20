---
title: Hadoop-HDFS-客户端部署（HDFS系列三）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-15 22:38:14
password:
summary: Hadoop之HDFS客户端部署
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS客户端相关

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