---
title: Hadoop-HDFS-IO流操作（HDFS系列五）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-20 22:53:57
password:
summary: HDFS相关IO流操作（自己编写api有关）
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS客户端相关

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

# 待续...

接下来是HDFS数据流相关讲解，据说是面试重点~

> 后天就可以看到屁屁了，开心~
> 希望明天bug全解决