---
title: Spark-SparkCore-键值对RDD数据分区器（SparkCore系列三）
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-03 21:34:42
password:
summary: 数据分区器（其实在RDD编程中已经有所涉猎）
tags:
- SparkCore
categories:
- 大数据
---

# SparkCore

## 键值对RDD数据分区器

Spark目前支持Hash分区和Range分区，用户也可以自定义分区，Hash分区为当前的默认分区，Spark中分区器直接决定了RDD中分区的个数、RDD中每条数据经过Shuffle过程属于哪个分区和Reduce的个数

**注意：**

(1)只有Key-Value类型的RDD才有分区器的，非Key-Value类型的RDD分区器的值是None

(2)每个RDD的分区ID范围：0~numPartitions-1，决定这个值是属于那个分区的。

### 获取RDD分区

可以通过使用RDD的partitioner 属性来获取 RDD 的分区方式。它会返回一个 scala.Option 对象， 通过get方法获取其中的值。相关源码如下：

```scala
def getPartition(key: Any): Int = key match {
  case null => 0
  case _ => Utils.nonNegativeMod(key.hashCode, numPartitions)
}

def nonNegativeMod(x: Int, mod: Int): Int = {
  val rawMod = x % mod
  rawMod + (if (rawMod < 0) mod else 0)
}
```

（1）创建一个pairRDD

```scala
scala> val pairs = sc.parallelize(List((1,1),(2,2),(3,3)))
——>
pairs: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[3] at parallelize at <console>:24
```

（2）查看RDD的分区器

```scala
scala> pairs.partitioner
——>
res1: Option[org.apache.spark.Partitioner] = None
```

（3）导入HashPartitioner类

```scala
scala> import org.apache.spark.HashPartitioner
import org.apache.spark.HashPartitioner
```

（4）使用HashPartitioner对RDD进行重新分区

```scala
scala> val partitioned = pairs.partitionBy(new HashPartitioner(2))
——>
partitioned: org.apache.spark.rdd.RDD[(Int, Int)] = ShuffledRDD[4] at partitionBy at <console>:27
```

（5）查看重新分区后RDD的分区器

```scala
scala> partitioned.partitioner
——>
res2: Option[org.apache.spark.Partitioner] = Some(org.apache.spark.HashPartitioner@2)
```

### Hash分区

HashPartitioner分区的原理：对于给定的key，计算其hashCode，并除以分区的个数取余，如果余数小于0，则用余数+分区的个数（否则加0），最后返回的值就是这个key所属的分区ID。

使用Hash分区的实操

```scala
scala> nopar.partitioner
——>
res1: Option[org.apache.spark.Partitioner] = None

scala> val nopar = sc.parallelize(List((1,3),(1,2),(2,4),(2,3),(3,6),(3,8)),8)
——>
nopar: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[10] at parallelize at <console>:24

scala>nopar.mapPartitionsWithIndex((index,iter)=>{ Iterator(index.toString+" : "+iter.mkString("|")) }).collect
——>
res2: Array[String] = Array("0 : ", 1 : (1,3), 2 : (1,2), 3 : (2,4), "4 : ", 5 : (2,3), 6 : (3,6), 7 : (3,8)) 

scala> val hashpar = nopar.partitionBy(new org.apache.spark.HashPartitioner(7))
——>
hashpar: org.apache.spark.rdd.RDD[(Int, Int)] = ShuffledRDD[12] at partitionBy at <console>:26

scala> hashpar.count
——>
res3: Long = 6

scala> hashpar.partitioner
——>
res4: Option[org.apache.spark.Partitioner] = Some(org.apache.spark.HashPartitioner@7)

scala> hashpar.mapPartitions(iter => Iterator(iter.length)).collect()
res19: Array[Int] = Array(0, 3, 1, 2, 0, 0, 0)
```

### Ranger分区

HashPartitioner分区弊端：可能导致每个分区中数据量的不均匀，极端情况下会导致某些分区拥有RDD的全部数据。

RangePartitioner作用：将一定范围内的数映射到某一个分区内，尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的，一个分区中的元素肯定都是比另一个分区内的元素小或者大，但是分区内的元素是不能保证顺序的。简单的说就是将一定范围内的数映射到某一个分区内。实现过程为：

第一步：先重整个RDD中抽取出样本数据，将样本数据排序，计算出每个分区的最大key值，形成一个Array[KEY]类型的数组变量rangeBounds；

第二步：判断key在rangeBounds中所处的范围，给出该key值在下一个RDD中的分区id下标；该分区器要求RDD中的KEY类型必须是可以排序的

### 自定义分区

要实现自定义的分区器，你需要继承 org.apache.spark.Partitioner 类并实现下面三个方法。 

（1）numPartitions：Int:返回创建出来的分区数。

（2）getPartition(key: Any)：Int:返回给定键的分区编号(0到numPartitions-1)。 

（3）equals()：Java 判断相等性的标准方法。这个方法的实现非常重要，Spark 需要用这个方法来检查你的分区器对象是否和其他分区器实例相同，这样 Spark 才可以判断两个 RDD 的分区方式是否相同。(这个方法某些情况下不写也是可以的)

使用自定义的 Partitioner 是很容易的：只要把它传给 partitionBy() 方法即可。Spark 中有许多依赖于数据混洗的方法，比如 join() 和 groupByKey()，它们也可以接收一个可选的 Partitioner 对象来控制输出数据的分区方式。

因为之前实现过自定义分区器，这里不在累述，demo地址

[com.swenchao.spark.Spark12_Oper11](https://github.com/Swenchao/SparkCode/blob/master/src/main/scala/com/swenchao/spark/Spark12_Oper11.scala)