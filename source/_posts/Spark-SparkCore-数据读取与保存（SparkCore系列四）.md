---
title: Spark-SparkCore-数据读取与保存（SparkCore系列四）
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-04 21:38:14
password:
summary: 数据读取与保存
tags:
- SparkCore
categories:
- 大数据
---

# SparkCore

## 数据读取与保存

Spark的数据读取及数据保存可以从两个维度来作区分：文件格式以及文件系统。

文件格式分为：text文件、json文件、csv文件、sequence文件以及object文件；

文件系统分为：本地文件系统、HDFS、HBASE以及数据库。

### 文件类数据读取与保存

#### Text文件

1）数据读取:textFile(String)

```scala
scala> val hdfsFile = sc.textFile("hdfs://hadoop102:9000/fruit.txt")
——>
hdfsFile: org.apache.spark.rdd.RDD[String] = hdfs://hadoop102:9000/fruit.txt MapPartitionsRDD[21] at textFile at <console>:24
```

2）数据保存: saveAsTextFile(String)

```scala
scala> hdfsFile.saveAsTextFile("/fruitOut")
```

#### Json文件

如果JSON文件中每一行就是一个JSON记录，那么可以通过将JSON文件当做文本文件来读取，然后利用相关的JSON库对每一条数据进行JSON解析。

**注意：**使用RDD读取JSON文件处理很复杂，同时SparkSQL集成了很好的处理JSON文件的方式，所以应用中多是采用SparkSQL处理JSON文件。

其中JSON文件地址：[in/user.json](https://github.com/Swenchao/SparkCode/blob/master/in/user.json)

```scala
package com.swenchao.spark

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
import scala.util.parsing.json.JSON

/**
 * @Author: Swenchao
 * @Date: 2020/10/03 下午 09:33
 * @Func: Json文件处理
 */
object Spark18_Json {
    def main(args: Array[String]): Unit = {

        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

        // 创建Spark上下文对象
        val sc: SparkContext = new SparkContext(conf)

        // 读取json文件
        val jsonRDD: RDD[String] = sc.textFile("in/user.json")

        // 进行解析
        val res: RDD[Option[Any]] = jsonRDD.map(JSON.parseFull)

        // 输出
        res.foreach(println)

        sc.stop()
    }
}
```

结果输出

```
    Some(Map(name -> zhangsan, age -> 18.0))
    Some(Map(name -> wangwu, age -> 18.0))
    Some(Map(name -> zhaoliu, age -> 18.0))
    Some(Map(name -> lisi, age -> 18.0))
```

**注：**这个包显示不能用了，但是结果还是出来了。从网上还没找到完全的替代品。

#### Sequence文件

SequenceFile文件是Hadoop用来存储二进制形式的key-value对而设计的一种平面文件(Flat File)。Spark 有专门用来读取 SequenceFile 的接口。在 SparkContext 中，可以调用 sequenceFile[ keyClass, valueClass](path)。

**注意：**SequenceFile文件只针对PairRDD

（1）创建一个RDD

```scala
scala> val rdd = sc.parallelize(Array((1,2),(3,4),(5,6)))
——>
rdd: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[13] at parallelize at <console>:24
```

（2）将RDD保存为Sequence文件

```scala
scala> rdd.saveAsSequenceFile("file:///opt/module/spark/seqFile")
```

（3）查看该文件

```shell
[user_test@hadoop102 seqFile]$ pwd
——>
/opt/module/spark/seqFile

[user_test@hadoop102 seqFile]$ ll
总用量 8
-rw-r--r-- 1 atguigu atguigu 108 10月  9 10:29 part-00000
-rw-r--r-- 1 atguigu atguigu 124 10月  9 10:29 part-00001
-rw-r--r-- 1 atguigu atguigu   0 10月  9 10:29 _SUCCESS

[user_test@hadoop102 seqFile]$ cat part-00000
——>
SEQ org.apache.hadoop.io.IntWritable org.apache.hadoop.io.IntWritable
```

（4）读取Sequence文件

```scala
scala> val seq = sc.sequenceFile[Int,Int]("file:///opt/module/spark/seqFile")
——>
seq: org.apache.spark.rdd.RDD[(Int, Int)] = MapPartitionsRDD[18] at sequenceFile at <console>:24
```

（5）打印读取后的Sequence文件

```scala
scala> seq.collect
——>
res1: Array[(Int, Int)] = Array((1,2), (3,4), (5,6))
```

#### 对象文件

对象文件是将对象序列化后保存的文件，采用Java的序列化机制。可以通过objectFile[k,v](path) 函数接收一个路径，读取对象文件，返回对应的 RDD，也可以通过调用saveAsObjectFile() 实现对对象文件的输出。因为是序列化所以要指定类型。

（1）创建一个RDD

```scala
scala> val rdd = sc.parallelize(Array(1,2,3,4))
——>
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[19] at parallelize at <console>:24
```

（2）将RDD保存为Object文件

```scala
scala> rdd.saveAsObjectFile("file:///opt/module/spark/objectFile")
```

（3）查看该文件

```shell
[user_test@hadoop102 objectFile]$ pwd
——>
/opt/module/spark/objectFile

[user_test@hadoop102 objectFile]$ ll
——>
总用量 8
-rw-r--r-- 1 atguigu atguigu 142 10月  9 10:37 part-00000
-rw-r--r-- 1 atguigu atguigu 142 10月  9 10:37 part-00001
-rw-r--r-- 1 atguigu atguigu   0 10月  9 10:37 _SUCCESS

[user_test@hadoop102 objectFile]$ cat part-00000 
——>
SEQ!org.apache.hadoop.io.NullWritable"org.apache.hadoop.io.BytesWritableW@`l
```

（4）读取Object文件

```scala
scala> val objFile = sc.objectFile[Int]("file:///opt/module/spark/objectFile")
——>
objFile: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[31] at objectFile at <console>:24
```

（5）打印读取后的Sequence文件

```scala
scala> objFile.collect
res19: Array[Int] = Array(1, 2, 3, 4)
```

### 文件系统类数据读取与保存

#### HDFS

Spark的整个生态系统与Hadoop是完全兼容的，所以对于Hadoop所支持的文件类型或者数据库类型，Spark也同样支持。另外，由于Hadoop的API有新旧两个版本，所以Spark为了能够兼容Hadoop所有的版本，也提供了两套创建操作接口。对于外部存储创建操作而言，hadoopRDD和newHadoopRDD是最为抽象的两个函数接口，主要包含以下四个参数。

1）输入格式(InputFormat)：制定数据输入的类型，如TextInputFormat等，新旧两个版本所引用的版本分别是 org.apache.hadoop.mapred.InputFormat 和 org.apache.hadoop.mapreduce.InputFormat(NewInputFormat)

2）键类型：指定[K,V]键值对中K的类型

3）值类型：指定[K,V]键值对中V的类型

4）分区值：指定由外部存储生成的RDD的partition数量的最小值,如果没有指定,系统会使用默认值defaultMinSplits

**注意：**其他创建操作的API接口都是为了方便最终的Spark程序开发者而设置的,是这两个接口的高效实现版本。例如，对于textFile而言，只有path这个指定文件路径的参数,其他参数在系统内部指定了默认值。

1. 在Hadoop中以压缩形式存储的数据,不需要指定解压方式就能够进行读取,因为Hadoop本身有一个解压器会根据压缩文件的后缀推断解压算法进行解压。

2. 如果用Spark从Hadoop中读取某种类型的数据不知道怎么读取的时候,上网查找一个使用map-reduce的时候是怎么读取这种这种数据的,然后再将对应的读取方式改写成上面的hadoopRDD和newAPIHadoopRDD两个类就行了

#### MySQL数据库连接

支持通过Java JDBC访问关系型数据库。需要通过JdbcRDD进行，示例:

表结构

[user]

| 列名 | 类型 |
| ---- | -------- |
| id | int(11) |
| name | varchar(255) |
| age | int(11) |

（1）添加依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.27</version>
</dependency>
```

（2）Mysql读取与新增：

demo地址：

[com.swenchao.spark.Spark19_Mysql](https://github.com/Swenchao/SparkCode/blob/master/src/main/scala/com/swenchao/spark/Spark19_Mysql.scala)

```scala
package com.swenchao.spark

import java.sql.{Connection, DriverManager, PreparedStatement}

import org.apache.spark.rdd.{JdbcRDD, RDD}
import org.apache.spark.{SparkConf, SparkContext}

/**
 * @Author: Swenchao
 * @Date: 2020/10/04 下午 09:33
 * @Func: Mysql连接
 */
object Spark19_Mysql {
    def main(args: Array[String]): Unit = {

        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")

        // 创建Spark上下文对象
        val sc: SparkContext = new SparkContext(conf)

        // 定义mysql参数
        val driver = "com.mysql.jdbc.Driver"
        val url = "jdbc:mysql://127.0.0.1:3306/rdd"
        val userName = "root"
        val passWd = "123456"

        /***************询数据*********************/
        //创建 JdbcRDD，访问数据库
//        val selectRDD = new JdbcRDD(
//            sc,
//            () => {
//                // 获取数据库连接对象
//                Class.forName(driver)
//                DriverManager.getConnection(url, userName, passWd)
//            },
//            "select name, age from user where id >= ? and id <= ?",
//            // 1是sql的第一个问号，2是sql的第二个问号
//            1,
//            3,
//            2,
//            (rs) => {
//                // sql中取的第一个是name ，取的第二个是age，所以是string 1， int 2
//                println(rs.getString(1), rs.getInt(2))
//            }
//        )
//
//        //打印最后结果
//        selectRDD.collect()

        /*******保存数据********/
        val saveRDD: RDD[(String, Int)] = sc.makeRDD(List(("韩七", 20), ("周八", 23), ("吴九", 17)), 2)

        // 此部分在executor中执行，所以要新增的三条数据不一定发给了谁，不一定谁先执行，因此其中的id可能不是这个顺序
//        saveRDD.foreach({
//            case (name, age) => {
//
//                // 新增多少条数据，就会创建多少个connection，所以效率很低。但是connection又不可以序列化，所以无法把这两句提到foreach外面
//                Class.forName(driver)
//                val connection: Connection = DriverManager.getConnection(url, userName, passWd)
//
//                val sql = "insert into user (name, age) values (?, ?)"
//
//                val statement: PreparedStatement = connection.prepareStatement(sql)
//                statement.setString(1, name)
//                statement.setInt(2, age)
//                statement.executeUpdate()
//
//                statement.close()
//                connection.close()
//            }
//        })

        /*******优化的新增数据***********/
        // 以分区为整体来建立与mysql的连接（一个分区用一个connection），但是由于以分区为单位来执行，因此可能会出现内存溢出
        saveRDD.foreachPartition(datas => {
            Class.forName(driver)
            val connection: Connection = DriverManager.getConnection(url, userName, passWd)
            datas.foreach({
                case (name, age) => {
                    val sql = "insert into user (name, age) values (?, ?)"
                    val statement: PreparedStatement = connection.prepareStatement(sql)
                    statement.setString(1, name)
                    statement.setInt(2, age)
                    statement.executeUpdate()

                    statement.close()
                }
            })
            connection.close()
        })

        sc.stop()
    }
}
```

#### HBase数据库

由于 org.apache.hadoop.hbase.mapreduce.TableInputFormat 类的实现，Spark 可以通过Hadoop输入格式访问HBase。这个输入格式会返回键值对数据，其中键的类型为org. apache.hadoop.hbase.io.ImmutableBytesWritable，而值的类型为org.apache.hadoop.hbase.client.
Result。

（1）添加依赖

```xml
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-server</artifactId>
	<version>1.3.1</version>
</dependency>

<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-client</artifactId>
	<version>1.3.1</version>
</dependency>
```

（2）从HBase读取数据

```scala
package com.atguigu

import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.client.Result
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.hadoop.hbase.util.Bytes

object HBaseSpark {

  def main(args: Array[String]): Unit = {

    //创建spark配置信息
    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("JdbcRDD")
    
    //创建SparkContext
    val sc = new SparkContext(sparkConf)
    
    //构建HBase配置信息
    val conf: Configuration = HBaseConfiguration.create()
    conf.set("hbase.zookeeper.quorum", "hadoop102,hadoop103,hadoop104")
    conf.set(TableInputFormat.INPUT_TABLE, "rddtable")
    
    //从HBase读取数据形成RDD
    val hbaseRDD: RDD[(ImmutableBytesWritable, Result)] = sc.newAPIHadoopRDD(
      conf,
      classOf[TableInputFormat],
      classOf[ImmutableBytesWritable],
      classOf[Result])
    
    val count: Long = hbaseRDD.count()
    println(count)
    
    //对hbaseRDD进行处理
    hbaseRDD.foreach {
      case (_, result) =>
        val key: String = Bytes.toString(result.getRow)
        val name: String = Bytes.toString(result.getValue(Bytes.toBytes("info"), Bytes.toBytes("name")))
        val color: String = Bytes.toString(result.getValue(Bytes.toBytes("info"), Bytes.toBytes("color")))
        println("RowKey:" + key + ",Name:" + name + ",Color:" + color)
    }
    
    //关闭连接
    sc.stop()
  }

}
```

3）往HBase写入

```scala
def main(args: Array[String]) {
//获取Spark配置信息并创建与spark的连接
  val sparkConf = new SparkConf().setMaster("local[*]").setAppName("HBaseApp")
  val sc = new SparkContext(sparkConf)

//创建HBaseConf
  val conf = HBaseConfiguration.create()
  val jobConf = new JobConf(conf)
  jobConf.setOutputFormat(classOf[TableOutputFormat])
  jobConf.set(TableOutputFormat.OUTPUT_TABLE, "fruit_spark")

//构建Hbase表描述器
  val fruitTable = TableName.valueOf("fruit_spark")
  val tableDescr = new HTableDescriptor(fruitTable)
  tableDescr.addFamily(new HColumnDescriptor("info".getBytes))

//创建Hbase表
  val admin = new HBaseAdmin(conf)
  if (admin.tableExists(fruitTable)) {
    admin.disableTable(fruitTable)
    admin.deleteTable(fruitTable)
  }
  admin.createTable(tableDescr)

//定义往Hbase插入数据的方法
  def convert(triple: (Int, String, Int)) = {
    val put = new Put(Bytes.toBytes(triple._1))
    put.addImmutable(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes(triple._2))
    put.addImmutable(Bytes.toBytes("info"), Bytes.toBytes("price"), Bytes.toBytes(triple._3))
    (new ImmutableBytesWritable, put)
  }

//创建一个RDD
  val initialRDD = sc.parallelize(List((1,"apple",11), (2,"banana",12), (3,"pear",13)))

//将RDD内容写到HBase
  val localData = initialRDD.map(convert)

  localData.saveAsHadoopDataset(jobConf)
}
```