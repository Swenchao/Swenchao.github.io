---
title: SparkSQL
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-06 16:38:40
password:
summary: SparkSQL完整笔记
tags:
- SparkSQL
categories:
- 大数据
---

# SparkSQL

## Spark SQL概述

### 什么是Spark SQL

Spark SQL是Spark用来处理结构化数据的一个模块，它提供了2个编程抽象：DataFrame和DataSet，并且作为分布式SQL查询引擎的作用（DataFrame当表来用，DataSet当对象来用）。

**注：**本来spark是没有数据结构的（最多是一个k-v，但是没有什么确定的name age之类的）

它是将Hive SQL转换成MapReduce然后提交到集群上执行，大大简化了编写MapReduc的程序的复杂性，由于MapReduce这种计算模型执行效率比较慢。所有Spark SQL的应运而生，它是将Spark SQL转换成RDD，然后提交到集群执行，执行效率非常快！

### Spark SQL的特点

1）易整合

2）统一的数据访问方式

3）兼容Hive

4）标准的数据连接

### 什么是DataFrame

与RDD类似，DataFrame也是一个分布式数据容器。然而DataFrame更像传统数据库的二维表格，除了数据以外，还记录数据的结构信息，即schema。同时，与Hive类似，DataFrame也支持嵌套数据类型（struct、array和map）。从API易用性的角度上看，DataFrame API提供的是一套高层的关系操作，比函数式的RDD API要更加友好。

![DataFrame和RDD的区别](1-1.png)

左侧的RDD[Person]虽然以Person为类型参数，但Spark框架本身不了解Person类的内部结构。而右侧的DataFrame却提供了详细的结构信息，使得Spark SQL可以清楚地知道该数据集中包含哪些列，每列的名称和类型各是什么。DataFrame是为数据提供了Schema的视图。可以把它当做数据库中的一张表来对待，DataFrame也是懒执行的。性能上比RDD要高，主要原因：

优化的执行计划：查询计划通过Spark catalyst optimiser进行优化。

```scala
users.join(events, users("id") === events("uid")).filter(events("date") > "2020-10-04")
```

![](1-2.png)

其中第一幅为我们自己写RDD时，所写的，可以实现合并过滤功能，但是中间会有很多无用的shuffle过程大大降低了效率。后面两幅是逐渐的优化。

图中构造了两个DataFrame，将它们join之后又做了一次filter操作。如果原封不动地执行这个执行计划，最终的执行效率是不高的。因为join是一个代价较大的操作，也可能会产生一个较大的数据集。如果我们能将filter下推到 join下方，先对DataFrame进行过滤，再join过滤后的较小的结果集，便可以有效缩短执行时间。而Spark SQL的查询优化器正是这样做的。简而言之，逻辑查询计划优化就是一个利用基于关系代数的等价变换，将高成本的操作替换为低成本操作的过程。 

### 什么是DataSet

1）是Dataframe API的一个扩展，是Spark最新的数据抽象。

2）用户友好的API风格，既具有类型安全检查也具有Dataframe的查询优化特性。

3）Dataset支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。

4）样例类被用来在Dataset中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。

5） Dataframe是Dataset的特列，DataFrame=Dataset[Row] ，所以可以通过as方法将Dataframe转换为Dataset。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息我都用Row来表示。

6）DataSet是强类型的。比如可以有Dataset[Car]，Dataset[Person].

7）DataFrame只是知道字段，但是不知道字段的类型，所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的，比如你可以对一个String进行减法操作，在执行的时候才报错，而DataSet不仅仅知道字段，而且知道字段类型，所以有更严格的错误检查。就跟JSON对象和类对象之间的类比。

![DataSet和DataFrame和RDD的关系](2-1.png)

RDD加上了关系就成了 DataFrame，DataFrame加上了类和属性就成了DataSet

## SparkSQL编程

### SparkSession新的起始点

在老的版本中，SparkSQL提供两种SQL查询起始点：一个叫SQLContext，用于Spark自己提供的SQL查询；一个叫HiveContext，用于连接Hive的查询。

SparkSession是Spark最新的SQL查询起始点，实质上是SQLContext和HiveContext的组合，所以在SQLContext和HiveContext上可用的API在SparkSession上同样是可以使用的。SparkSession内部封装了sparkContext，所以计算实际上是由sparkContext完成的。

### DataFrame

#### 创建

在Spark SQL中SparkSession是创建DataFrame和执行SQL的入口，创建DataFrame有三种方式：通过Spark的数据源进行创建；从一个存在的RDD进行转换；还可以从Hive Table进行查询返回。

1）从Spark数据源进行创建

（1）查看Spark数据源进行创建的文件格式

```scala
scala> spark.read.
——>
// 可读取文件格式
csv   format   jdbc   json   load   option   options   orc   parquet   schema   table   text   textFile
```

（2）读取json文件创建DataFrame

```scala
scala> val df = spark.read.json("file///opt/module/spark/data/input/user.json")
——>
// 直接得到了DataFrame并获得了字段
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
```

（3）展示结果

```scala
scala> df.show
——>
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
```

2）从RDD进行转换

后面用到细说

3）从Hive Table进行查询返回

hive数据库那节会细说

#### SQL风格语法(主要)

1）创建一个DataFrame

```scala
scala> val df = spark.read.json("file///opt/module/spark/data/input/user.json")
——>
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
```

2）对DataFrame创建一个临时视图（之所以叫临时视图不叫临时表是因为RDD是不可变的，表是可变的，视图是不可变的）就可以用sql来进行查询了

```scala
scala> df.createOrReplaceTempView("student")
```

3）通过SQL语句实现查询全表

```scala
scala> val sqlDF = spark.sql("select * from student")
——>
sqlDF: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
```

4）结果展示

```scala
scala> sqlDF.show
——>
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala> spark.sql("select age from student").show
——>
+----+
| age|
+----+
|null|
|  30|
|  19|
+----+

scala> spark.sql("select avg(age) from student").show
——>
+---------+
| avg(age)|
+---------+
|     20.0|
+---------+
```

注意：临时表是Session范围内的，Session退出后，表就失效了。如果想应用范围内有效，可以使用全局表。注意使用全局表时需要全路径访问，如：global_temp.student

5）对于DataFrame创建一个全局表

```scala
scala> df.createGlobalTempView("student")
```

6）通过SQL语句实现查询全表

```scala
scala> spark.sql("SELECT * FROM global_temp.student").show()
——>
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala> spark.newSession().sql("SELECT * FROM global_temp.student").show()
——>
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
```

#### DSL风格语法(次要)

1）创建一个DateFrame

```scala
scala> spark.read.
——>
csv   format   jdbc   json   load   option   options   orc   parquet   schema   table   text   textFile
```

2）查看DataFrame的Schema信息

```scala
scala> df.printSchema
——>
root
 |-- age: long (nullable = true)
 |-- name: string (nullable = true)
```

3）只查看”name”列数据

```scala
scala> df.select("name").show()
——>
+-------+
|   name|
+-------+
|Michael|
|   Andy|
| Justin|
+-------+
```

4）查看”name”列数据以及”age+1”数据

```scala
scala> df.select($"name", $"age" + 1).show()
——>
+-------+---------+
|   name|(age + 1)|
+-------+---------+
|Michael|     null|
|   Andy|       31|
| Justin|       20|
+-------+---------+
```

5）查看”age”大于”21”的数据

```scala
scala> df.filter($"age" > 21).show()
——>
+---+----+
|age|name|
+---+----+
| 30|Andy|
+---+----+
```

6）按照”age”分组，查看数据条数

```scala
scala> df.groupBy("age").count().show()
——>
+----+------+
| age| count|
+----+------+
|  19|     1|
|null|     1|
|  30|     1|
+----+------+
```

#### RDD转换为DateFrame

**注意：**如果需要RDD与DF或者DS之间操作，那么都需要引入 import spark.implicits\._  (spark不是包名，而是sparkSession对象的名称)

前置条件：导入隐式转换并创建一个RDD

```scala
scala> import spark.implicits._
——>
import spark.implicits._

scala> val rdd = sc.makeRDD(List((1, "zhangsan", 20), (2, "lisi", 30), (3, "wangwu", 40)))
——>
rdd: org.apache.spark.rdd.RDD[(Int, String, Int)] = ParallelCollectionRDD[0] at makeRDD at <console>:24
```

1）通过手动确定转换

```scala
scala> val df =rdd.toDF("id", "name", "age")
——>
df: org.apache.spark.sql.DataFrame = [id: int, name: string ... 1 more field]

scala> df.show
——>
+---+--------+---+
| id|    name|age|
+---+--------+---+
|  1|zhangsan| 20|
|  2|    lisi| 30|
|  3|  wangwu| 40|
+---+--------+---+
```

2）通过反射确定（需要用到样例类）

（1）创建一个样例类

```scala
scala> case class People(name:String, age:Int)
——>
defined class People
```

（2）根据样例类将RDD转换为DataFrame

```scala
scala> val rdd = sc.makeRDD(List(("zhangsan", 20), ("lisi", 30), ("wangwu", 40)))
——>
rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[4] at makeRDD at <console>:24

scala> val peopleRDD = rdd.map(t=>{People(t._1, t._2)})
——>
peopleRDD: org.apache.spark.rdd.RDD[People] = MapPartitionsRDD[5] at map at <console>:28

scala> peopleRDD.toDF
——>
res2: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> val df = peopleRDD.toDF
——>
df: org.apache.spark.sql.DataFrame = [name: string, age: int]

scala> df.show
——>
+--------+---+
|    name|age|
+--------+---+
|zhangsan| 20|
|    lisi| 30|
|  wangwu| 40|
+--------+---+
```

3）通过编程的方式（了解）

（1）导入所需的类型

```scala
scala> import org.apache.spark.sql.types._
——>
import org.apache.spark.sql.types._
```

（2）创建Schema

```scala
scala> val structType: StructType = StructType(StructField("name", StringType) :: StructField("age", IntegerType) :: Nil)
——>
structType: org.apache.spark.sql.types.StructType = StructType(StructField(name,StringType,true), StructField(age,IntegerType,true))
```

（3）导入所需的类型

```scala
scala> import org.apache.spark.sql.Row
import org.apache.spark.sql.Row
```

（4）根据给定的类型创建二元组RDD

```scala
scala> val data = peopleRDD.map{ x => val para = x.split(",");Row(para(0),para(1).trim.toInt)}
data: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[6] at map at <console>:33
```

（5）根据数据及给定的schema创建DataFrame

```scala
scala> val dataFrame = spark.createDataFrame(data, structType)
dataFrame: org.apache.spark.sql.DataFrame = [name: string, age: int]
```

#### DateFrame转换为RDD

直接调用rdd即可

1）创建一个DataFrame

```scala
scala> val df = spark.read.json("/opt/module/spark/examples/src/main/resources/people.json")
——>
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
```

2）将DataFrame转换为RDD

```scala
scala> val dfToRDD = df.rdd
dfToRDD: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[19] at rdd at <console>:29
```

3）打印RDD

```scala
scala> dfToRDD.collect
res13: Array[org.apache.spark.sql.Row] = Array([Michael, 29], [Andy, 30], [Justin, 19])
```

### DataSet

Dataset是具有强类型的数据集合，需要提供对应的类型信息。

#### 创建

1）创建一个样例类

```scala
scala> case class Person(name: String, age: Long)
——>
defined class Person
```

2）创建DataSet

```scala
scala> val caseClassDS = Seq(Person("Andy", 32)).toDS()
——>
caseClassDS: org.apache.spark.sql.Dataset[Person] = [name: string, age: bigint]
```

3）查看

```scala
scala> caseClassDS.show
——>
+----+---+
|name|age|
+----+---+
|Andy| 32|
+----+---+
```

#### RDD转换为DataSet

SparkSQL能够自动将包含有case类的RDD转换成DataFrame，case类定义了table的结构，case类属性通过反射变成了表的列名。

1）创建一个RDD

```scala
scala> val rdd = sc.makeRDD(List(("zhangsan", 20), ("lisi", 30), ("wangwu", 40)))
——>
rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[4] at makeRDD at <console>:24

```

2）创建一个样例类

```scala
scala> case class Person(name: String, age: Long)
——>
defined class Person
```

3）将RDD转化为DataSet

```scala
scala> val mapRDD = rdd.map(t => {Person(t._1, t._2)})
——>
mapRDD: org.apache.spark.rdd.RDD[Person] = MapPartitionsRDD[10] at map at <console>:28

scala> mapRDD.toDS.show
——>
+--------+---+
|    name|age|
+--------+---+
|zhangsan| 20|
|    lisi| 30|
|  wangwu| 40|
+--------+---+
```

#### DataSet转换为RDD

调用rdd方法即可。

1）创建一个DataSet

```scala
scala> val DS = Seq(Person("Andy", 32)).toDS()
——>
DS: org.apache.spark.sql.Dataset[Person] = [name: string, age: bigint]
```

2）将DataSet转换为RDD

```scala
scala> DS.rdd
——>
res11: org.apache.spark.rdd.RDD[Person] = MapPartitionsRDD[15] at rdd at <console>:28
```

### DataFrame与DataSet的互操作

1. DataFrame转换为DataSet

1）创建一个DateFrame

```scala
scala> val df = spark.read.json("examples/src/main/resources/people.json")
——>
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
```

2）创建一个样例类

```scala
scala> case class Person(name: String, age: Long)
——>
defined class Person
```

3）将DateFrame转化为DataSet

```scala
scala> df.as[Person]
——>
res14: org.apache.spark.sql.Dataset[Person] = [age: bigint, name: string]
```

2. DataSet转换为DataFrame

1）创建一个样例类

```scala
scala> case class Person(name: String, age: Long)
——>
defined class Person
```

2）创建DataSet

```scala
scala> val ds = Seq(Person("Andy", 32)).toDS()
——>
ds: org.apache.spark.sql.Dataset[Person] = [name: string, age: bigint]
```

3）将DataSet转化为DataFrame

```scala
scala> val df = ds.toDF
——>
df: org.apache.spark.sql.DataFrame = [name: string, age: bigint]
```

4）展示

```scala
scala> df.show
——>
+----+---+
|name|age|
+----+---+
|Andy| 32|
+----+---+
```

#### DataSet转DataFrame

这个很简单，因为只是把case class封装成Row

（1）导入隐式转换

```scala
import spark.implicits._
```

（2）转换

```scala
val testDF = testDS.toDF
```

#### DataFrame转DataSet

（1）导入隐式转换

```scala
import spark.implicits._
````

（2）创建样例类

```scala
case class Coltest(col1:String,col2:Int)extends Serializable //定义字段名和类型
```

（3）转换

```scala
val testDS = testDF.as[Coltest]
```

这种方法就是在给出每一列的类型后，使用as方法，转成Dataset，这在数据类型是DataFrame又需要针对各个字段处理时极为方便。在使用一些特殊的操作时，一定要加上 import spark.implicits\._ 不然toDF、toDS无法使用。

![](2-2.png)

RDD加上结构就是DF，DF加上类型就是DS ——> RDD加上类就是DS

### RDD、DataFrame、DataSet

在SparkSQL中Spark为我们提供了两个新的抽象，分别是DataFrame和DataSet。

DF、DS和RDD区别：

版本的产生：RDD (Spark1.0) —> Dataframe(Spark1.3) —> Dataset(Spark1.6)

如果同样的数据都给到这三个数据结构，他们分别计算之后，都会给出相同的结果。不同是的他们的执行效率和执行方式。

在后期的Spark版本中，DataSet会逐步取代RDD和DataFrame成为唯一的API接口。

#### 三者的共性

1. RDD、DataFrame、Dataset全都是spark平台下的分布式弹性数据集，为处理超大型数据提供便利

2. 三者都有惰性机制，在进行创建、转换，如map方法时，不会立即执行，只有在遇到Action如foreach时，三者才会开始遍历运算。

3. 三者都会根据spark的内存情况自动缓存运算，这样即使数据量很大，也不用担心会内存溢出。

4. 三者都有partition的概念

5. 三者有许多共同的函数，如filter，排序等

6. 在对DataFrame和Dataset进行操作许多操作都需要这个包进行支持
   import spark.implicits._

7. DataFrame和Dataset均可使用模式匹配获取各个字段的值和类型

#### 三者的区别

1. RDD:

1）RDD一般和spark mlib（机器学习库）同时使用

2）RDD不支持sparksql操作

2. DataFrame:

1）与RDD和Dataset不同，DataFrame每一行的类型固定为Row，每一列的值没法直接访问，只有通过解析才能获取各个字段的值，如：

```scala
testDF.foreach{
    line =>
    val col1=line.getAs[String]("col1")
    val col2=line.getAs[String]("col2")
}
```

2）DataFrame与Dataset一般不与spark mlib同时使用

3）DataFrame与Dataset均支持sparksql的操作，比如select，groupby之类，还能注册临时表/视窗，进行sql语句操作，如：

```scala
dataDF.createOrReplaceTempView("tmp")
spark.sql("select  ROW,DATE from tmp where DATE is not null order by DATE").show(100,false)
```

4）DataFrame与Dataset支持一些特别方便的保存方式，比如保存成csv，可以带上表头，这样每一列的字段名一目了然

```scala
//保存
val saveoptions = Map("header" -> "true", "delimiter" -> "\t", "path" -> "hdfs://hadoop102:9000/test")
datawDF.write.format("com.atguigu.spark.csv").mode(SaveMode.Overwrite).options(saveoptions).save()
//读取
val options = Map("header" -> "true", "delimiter" -> "\t", "path" -> "hdfs://hadoop102:9000/test")
val datarDF= spark.read.options(options).format("com.atguigu.spark.csv").load()
```

利用这样的保存方式，可以方便的获得字段名和列的对应，而且分隔符（delimiter）可以自由指定。

3. Dataset:

1）Dataset和DataFrame拥有完全相同的成员函数，区别只是每一行的数据类型不同。

2）DataFrame也可以叫Dataset[Row],每一行的类型是Row，不解析，每一行究竟有哪些字段，各个字段又是什么类型都无从得知，只能用上面提到的getAS方法或者共性中的第七条提到的模式匹配拿出特定字段。而Dataset中，每一行是什么类型是不一定的，在自定义了case class之后可以很自由的获得每一行的信息

```scala
case class Coltest(col1:String,col2:Int)extends Serializable //定义字段名和类型
/**
 rdd
 ("a", 1)
 ("b", 1)
 ("a", 1)
**/
val test: Dataset[Coltest]=rdd.map{line=>
      Coltest(line._1,line._2)
    }.toDS
test.map{
      line=>
        println(line.col1)
        println(line.col2)
    }
```

可以看出，Dataset在需要访问列中的某个字段时是非常方便的，然而，如果要写一些适配性很强的函数时，如果使用Dataset，行的类型又不确定，可能是各种case class，无法实现适配，这时候用DataFrame即Dataset[Row]就能比较好的解决问题

### IDEA创建SparkSQL程序

IDEA中程序的打包和运行方式都和SparkCore类似，Maven依赖中需要添加新的依赖项：

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.11</artifactId>
    <version>2.1.1</version>
</dependency>
```

程序如下：

```scala
package com.swenchao.spark.sql

import org.apache.spark.sql.{DataFrame, SparkSession}
import org.apache.spark.SparkConf

/**
 * @Author: Swenchao
 * @Date: 2020/10/5 下午 04:38
 * @Description:
 * @Modified:
 * @Version:
 */
object SparkSQL01_demo {
    def main(args: Array[String]): Unit = {

        // 创建配置对象
        val sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkSQL01_demo")

        // 创建SparkSession
        val spark: SparkSession = SparkSession.builder().config(sparkConf).getOrCreate()

        // 读取文件数据
        val frame: DataFrame = spark.read.json("in/user.json")

        // 展示
        frame.show()

        spark.stop
    }
}
```

### 用户自定义函数

在Shell窗口中可以通过spark.udf功能用户可以自定义函数。

#### 用户自定义UDF函数

```scala
scala> val df = spark.read.json("file///opt/module/spark/data/input/user.json")
——>
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]

scala> df.show()
——>
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala> spark.udf.register("addName", (x:String)=> "Name:"+x)
——>
res5: org.apache.spark.sql.expressions.UserDefinedFunction = UserDefinedFunction(<function1>,StringType,Some(List(StringType)))

scala> df.createOrReplaceTempView("people")

scala> spark.sql("Select addName(name), age from people").show()
——>
+-----------------+----+
|UDF:addName(name)| age|
+-----------------+----+
|     Name:Michael|null|
|        Name:Andy|  30|
|      Name:Justin|  19|
+-----------------+----+
```

#### 用户自定义聚合函数

强类型的Dataset和弱类型的DataFrame都提供了相关的聚合函数， 如 count()，countDistinct()，avg()，max()，min()。除此之外，用户可以设定自己的自定义聚合函数。

弱类型用户自定义聚合函数：通过继承UserDefinedAggregateFunction来实现用户自定义聚合函数。下面展示一个求平均工资的自定义聚合函数。

demo地址：

[com/swenchao/spark/sql/SparkSQL05_UDAF.scala](https://github.com/Swenchao/SparkCode/blob/master/src/main/scala/com/swenchao/spark/sql/SparkSQL05_UDAF.scala)

[com/swenchao/spark/sql/SparkSQL06_UDAF_Class.scala](https://github.com/Swenchao/SparkCode/blob/master/src/main/scala/com/swenchao/spark/sql/SparkSQL06_UDAF_Class.scala)


## SparkSQL数据源

### 通用加载/保存方法

#### 手动指定选项

Spark SQL的DataFrame接口支持多种数据源的操作。一个DataFrame可以进行RDDs方式的操作，也可以被注册为临时表。把DataFrame注册为临时表之后，就可以对该DataFrame执行SQL查询。

Spark SQL的默认数据源为Parquet格式。数据源为Parquet文件时，Spark SQL可以方便的执行所有的操作。修改配置项 spark.sql.sources.default，可修改默认数据源格式。

```scala
scala> val df = spark.read.load("examples/src/main/resources/users.parquet") df.select("name", "favorite_color").write.save("namesAndFavColors.parquet")
```

当数据源格式不是parquet格式文件时，需要手动指定数据源的格式。数据源格式需要指定全名（例如：org.apache.spark.sql.parquet），如果数据源格式为内置格式，则只需要指定简称定json, parquet, jdbc, orc, libsvm, csv, text来指定数据的格式。

可以通过 SparkSession 提供的 read.load 方法用于通用加载数据，使用write和save保存数据。 

```scala
scala> val peopleDF = spark.read.format("json").load("examples/src/main/resources/people.json")
——>
peopleDF.write.format("parquet").save("hdfs://hadoop102:9000/namesAndAges.parquet")
```

除此之外，可以直接运行SQL在文件上：

```scala
scala> val sqlDF = spark.sql("SELECT * FROM parquet.`hdfs://hadoop102:9000/namesAndAges.parquet`")

scala> sqlDF.show()

scala> val peopleDF = spark.read.format("json").load("examples/src/main/resources/people.json")
——>
peopleDF: org.apache.spark.sql.DataFrame = [age: bigint, name: string]

scala> peopleDF.write.format("parquet").save("hdfs://hadoop102:9000/namesAndAges.parquet")

scala> peopleDF.show()
——>
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala> val sqlDF = spark.sql("SELECT * FROM parquet.`hdfs:// hadoop102:9000/namesAndAges.parquet`")

scala> sqlDF.show()
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
```

#### 文件保存选项

可以采用SaveMode执行存储操作，SaveMode定义了对数据的处理模式。需要注意的是，这些保存模式不使用任何锁定，不是原子操作。此外，当使用Overwrite方式执行时，在输出新数据之前原数据就已经被删除。SaveMode详细介绍如下表：

| Scala/Java                      | Any Language     | Meaning              |
| ------------------------------- | ---------------- | -------------------- |
| SaveMode.ErrorIfExists(default) | "error"(default) | 如果文件存在，则报错 |
| SaveMode.Append                 | "append"         | 追加                 |
| SaveMode.Overwrite              | "overwrite"      | 覆写                 |
| SaveMode.Ignore                 | "ignore"         | 数据存在，则忽略     |

### JSON文件

Spark SQL 能够自动推测 JSON数据集的结构，并将它加载为一个Dataset[Row]. 可以通过SparkSession.read.json()去加载一个 一个JSON 文件。

**注意：**这个JSON文件不是一个传统的JSON文件，每一行都得是一个JSON串。

```
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

### Parquet文件

Parquet是一种流行的列式存储格式，可以高效地存储具有嵌套字段的记录。Parquet格式经常在Hadoop生态圈中被使用，它也支持Spark SQL的全部数据类型。Spark SQL 提供了直接读取和存储 Parquet 格式文件的方法。 

### JDBC

Spark SQL可以通过JDBC从关系型数据库中读取数据的方式创建DataFrame，通过对DataFrame一系列的计算后，还可以将数据再写回关系型数据库中。

**注意：**需要将相关的数据库驱动放到spark的类路径下。

（1）启动spark-shell

```shell
$ bin/spark-shell
```

（2）从Mysql数据库加载数据方式一

```scala
val jdbcDF = spark.read.format("jdbc").option("url", "jdbc:mysql://hadoop102:3306/rdd").option("dbtable", "rddtable").option("user", "root").option("password", "123456").load()
```

（3）从Mysql数据库加载数据方式二

```scala
val connectionProperties = new Properties()

connectionProperties.put("user", "root")

connectionProperties.put("password", "123456")

val jdbcDF2 = spark.read.jdbc("jdbc:mysql://hadoop102:3306/rdd", "rddtable", connectionProperties)
```

（4）将数据写入Mysql方式一

```scala
jdbcDF.write.format("jdbc").option("url", "jdbc:mysql://hadoop102:3306/rdd").option("dbtable", "dftable").option("user", "root").option("password", "123456").save()
```

（5）将数据写入Mysql方式二

```scala
jdbcDF2.write.jdbc("jdbc:mysql://hadoop102:3306/rdd", "db", connectionProperties)
```

### Hive数据库

Apache Hive是Hadoop上的SQL引擎，Spark SQL编译时可以包含Hive支持，也可以不包含。包含Hive支持的Spark SQL可以支持Hive表访问、UDF(用户自定义函数)以及 Hive 查询语言(HiveQL/HQL)等。需要强调的一点是，如果要在Spark SQL中包含Hive的库，并不需要事先安装Hive。一般来说，最好还是在编译Spark SQL时引入Hive支持，这样就可以使用这些特性了。如果你下载的是二进制版本的 Spark，它应该已经在编译时添加了 Hive 支持。 

若要把Spark SQL连接到一个部署好的Hive上，你必须把hive-site.xml复制到 Spark的配置文件目录中($SPARK_HOME/conf)。即使没有部署好Hive，Spark SQL也可以运行。 需要注意的是，如果你没有部署好Hive，Spark SQL会在当前的工作目录中创建出自己的Hive 元数据仓库，叫作 metastore_db。此外，如果你尝试使用 HiveQL 中的 CREATE TABLE (并非 CREATE EXTERNAL TABLE)语句来创建表，这些表会被放在你默认的文件系统中的 /user/hive/warehouse 目录中(如果你的 classpath 中有配好的 hdfs-site.xml，默认的文件系统就是 HDFS，否则就是本地文件系统)。

#### 内嵌Hive应用

如果要使用内嵌的Hive，什么都不用做，直接用就可以了。 

可以通过添加参数初次指定数据仓库地址：

```xml
--conf spark.sql.warehouse.dir=hdfs://hadoop102/spark-wearhouse
```

![](3-1.png)

**注意：**如果你使用的是内部的Hive，在Spark2.0之后，spark.sql.warehouse.dir用于指定数据仓库的地址，如果你需要是用HDFS作为路径，那么需要将core-site.xml和hdfs-site.xml 加入到Spark conf目录，否则只会创建master节点上的warehouse目录，查询时会出现文件找不到的问题，这是需要使用HDFS，则需要将metastore删除，重启集群。

#### 外部Hive应用

如果想连接外部已经部署好的Hive，需要通过以下几个步骤。

1) 将Hive中的hive-site.xml拷贝或者软连接到Spark安装目录下的conf目录下。

2) 打开spark shell，注意带上访问Hive元数据库的JDBC客户端

```shell
$ bin/spark-shell  --jars mysql-connector-java-5.1.27-bin.jar
```

3) 运行结果

![](3-2.png)

外部hive创建的表

#### 运行Spark SQL CLI

Spark SQL CLI可以很方便的在本地运行Hive元数据服务以及从命令行执行查询任务。在Spark目录下执行如下命令启动Spark SQL CLI：

```scala
./bin/spark-sql
```

进入另外一个sparkSQL的界面，可以直接写sql，类似于mysql的黑白框

#### 代码中使用Hive

（1）添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-hive -->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-hive_2.11</artifactId>
    <version>2.1.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>1.2.1</version>
</dependency>
```

（2）创建SparkSession时需要添加hive支持（.enableHiveSupport() 这部分）

```scala
// 使用内置Hive需要指定一个Hive仓库地址（config("spark.sql.warehouse.dir", warehouseLocation) 这部分类似）
val warehouseLocation: String = new File("spark-warehouse").getAbsolutePath

val spark = SparkSession.builder().appName("Spark Hive Example").config("spark.sql.warehouse.dir", warehouseLocation).enableHiveSupport().getOrCreate()
```

**注意：**若使用的是外部Hive，则需要将hive-site.xml添加到ClassPath下。