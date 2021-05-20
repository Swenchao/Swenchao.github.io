---
title: Hadoop-MapReduce-概述（MapReduce系列一）
top: false
cover: true
toc: true
mathjax: true
date: 2020-09-06 17:30:14
password:
summary: 从今天开始Mapreduce了。MapReduce概述
tags:
- Hadoop-MapReduce
categories:
- 大数据
---

# MapReduce

## MapReduce概述

### MapReduce定义

![](1.png)

### MapReduce优缺点

优点

![](2.png)

![](3.png)

缺点

![](4.png)

### MapReduce核心思想

![](5.png)

分布式的运算程序往往需要分成至少2个阶段。

1）第一个阶段的MapTask并发实例，完全并行运行，互不相干。

2）第二个阶段的ReduceTask并发实例互不相干，但是他们的数据依赖于上一个阶段的所有MapTask并发实例的输出。

3）MapReduce编程模型只能包含一个Map阶段和一个Reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序，串行运行(前一个MapReduce模型输出作为后一个MapReduce模型的输入)。

总结：分析WordCount数据流走向深入理解MapReduce核心思想。

### MapReduce进程

![](6.png)

通俗点讲，AppMaster相当于一个job的老大，ResourceManager相当于所有的老大，NodeManager相当于单个节点的老大。

### 官方WordCount源码

采用反编译工具反编译源码，发现WordCount案例有Map类、Reduce类和驱动类。且数据的类型是Hadoop自身封装的序列化类型。

### 常用数据序列化类型

|  Java类型   | Hadoop Write 类型 |
|  ----  | ----  |
| boolean | BooleanWritable |
| byte | ByteWritable |
| int |	IntWritable |
| float	| FloatWritable |
| long | LongWritable |
| double | DoubleWritable |
| String | Text |
| map | MapWritable |
| array | ArrayWritable |

### MapReduce编程规范

用户编写的程序分成三个部分：Mapper、Reducer和Driver。

![](7.png)

![](8.png)

![](9.png)

### WordCount案例实操

1．需求

在给定的文本文件中统计输出每一个单词出现的总次数

2．需求分析

按照MapReduce编程规范，分别编写Mapper，Reducer，Driver

Mapper，Reducer工作已经了解，下面是Driver工作

![](9.png)

3．环境准备

（1）创建maven工程

（2）在pom.xml文件中添加如下依赖

```xml
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
</dependencies>
```

（2）在项目的src/main/resources目录下，新建一个文件，命名为“log4j.properties”，在文件中填入。

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

4．编写程序
（1）编写Mapper类

LongWritable：KEYIN 输入数据的key（读取游标）
Text：VALUEIN 输入数据value（输入文本）
输出数据类型 <sc, 1>, <ss, 2>...
Text：KEYOUT 输出数据key类型
IntWritable：VALUEOUT 输出数据value类型

```java
package com.swenchao.mr.wordcount;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordcountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	
	Text k = new Text();
	IntWritable v = new IntWritable(1);
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
		
		// 1 获取一行
		String line = value.toString();
		
		// 2 切割
		String[] words = line.split(" ");
		
		// 3 输出
		for (String word : words) {
			
			k.set(word);
			context.write(k, v);
		}
	}
}
```

（2）编写Reducer类

map阶段输出的kv

Text：KEYIN
IntWritable： VALUEIN

```java
package com.swenchao.mr.wordcount;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordcountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{

int sum;
IntWritable v = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {
		
		// 1 累加求和
		sum = 0;
		for (IntWritable count : values) {
			sum += count.get();
		}
		
		// 2 输出
       v.set(sum);
		context.write(key,v);
	}
}
```

（3）编写Driver驱动类

```java
package com.swenchao.mr.wordcount;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordcountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

		// 1 获取配置信息以及封装任务
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 设置jar加载路径
		job.setJarByClass(WordcountDriver.class);

		// 3 设置map和reduce类
		job.setMapperClass(WordcountMapper.class);
		job.setReducerClass(WordcountReducer.class);

		// 4 设置map输出
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);

		// 5 设置最终输出kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		// 6 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// 7 提交
		boolean result = job.waitForCompletion(true);

		System.exit(result ? 0 : 1);
	}
}
```

5．本地测试

（1）如果电脑系统是win7的就将win7的hadoop jar包解压到非中文路径，并在Windows环境上配置HADOOP_HOME环境变量。如果是电脑win10操作系统，就解压win10的hadoop jar包，并配置HADOOP_HOME环境变量。

**注意：win8电脑和win10家庭版操作系统可能有问题，需要重新编译源码或者更改操作系统。**

（2）在Idea上运行程序

注意在运行添加arguments时，输出路径(args[1])要定位到一个现在还没有的**路径**，而不是一个文件（比如：d:/scwri/Desktop/output，其中output是现在并不存在的一个路径）

6．集群上测试

（0）用maven打jar包，需要添加的打包插件依赖

注意：标记红颜色的部分需要替换为自己工程主类

```xml
<build>
	<plugins>
		<plugin>
			<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin </artifactId>
				<configuration>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
							<mainClass>com.atguigu.mr.WordcountDriver</mainClass>
						</manifest>
					</archive>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```

如果maven修改后，有红叉。则可以将maven设置成自动导入，点击file再点settings，然后搜多maven 如下：

![](10.png)

（1）将程序打成jar包，然后拷贝到Hadoop集群中

[exlipse]

步骤：右键 -> Run as -> maven install。等待编译完成就会在项目的target文件夹中生成jar包。如果看不到。在项目上右键-》Refresh，即可看到。修改不带依赖的jar包名称为wc.jar，并拷贝该jar包到Hadoop集群。

[idea]

步骤：右键项目 -> Run Maven -> install。然后就跟上面一样的操作。

（2）启动Hadoop集群

（3）执行WordCount程序

```shell
[user_test@hadoop102 hadoop-2.7.2]$ hadoop jar  wc.jar com.swenchao.mr.wordcount.WordcountDriver /user/test/input/test.txt /user/atguigu/output
```

**注：**其中输出路径（/user/atguigu/output）必须是之前不存在的；输入文件（test.txt）是已经上传的。

