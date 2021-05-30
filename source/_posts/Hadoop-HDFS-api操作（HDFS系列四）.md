---
title: Hadoop-HDFS-api操作（HDFS系列四）
top: true
cover: true
toc: true
mathjax: true
date: 2020-08-16 22:52:16
password:
summary: Hadoop之HDFS相关api使用
tags:
- Hadoop-HDFS
categories:
- 大数据
---

# HDFS

## HDFS客户端相关

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