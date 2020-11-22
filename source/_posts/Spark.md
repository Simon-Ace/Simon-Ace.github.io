---
type: blog
title: Spark
date: 2020-11-05
categories: 教程
tags: Spark, 教程
cover: false
meta:
  date: false
---

# Spark

## 一、SparkCore

### RDD 创建

- 从集合中创建

```java
val listRdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4))
listRdd.foreach(println)

val arrayRDD: RDD[Int] = sc.parallelize(Array(1, 2, 3, 4))
arrayRDD.foreach(println)
```



- 由外部存储系统的数据集创建

```java
 val lines: RDD[String] = sc.textFile("in")
```

### RDD 转换算子

#### Value 类型

##### map

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 10)
val mulRdd: RDD[Int] = listRdd.map(_ * 2)
mulRdd.collect().foreach(println)
```

##### mapPartitions

对每一个分区中的数据批处理。相当于只给每个分区的数据，只发送一次计算；而 map 的实现会给每个数据发送一次计算，增加了网络传输消耗；但是 mapPartitions 由于以整个分区为单位，可能会造成 OOM

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 10)
val mapParRdd: RDD[Int] = listRdd.mapPartitions(datas => {
  datas.map(_ * 2)
})
mapParRdd.collect().foreach(println)
```

##### mapPartitionsWithIndex

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 10,3)
val tupleRdd: RDD[(Int, String)] = listRdd.mapPartitionsWithIndex {
  case (num, datas) => {
    datas.map((_, "partition_num: " + num))
  }
}
tupleRdd.collect().foreach(println)
```

##### flatMap

扁平化，变成一个一个单独的元素

```java
val listRdd: RDD[List[Int]] = sc.makeRDD(Array(List(1, 2), List(3, 4)))
val flatRdd: RDD[Int] = listRdd.flatMap(datas => datas)
flatRdd.collect().foreach(println)
```

##### glom

将同一个分区的元素，放到一个数组里

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 16, 4)
val glomRdd: RDD[Array[Int]] = listRdd.glom()
glomRdd.collect().foreach(array => {
  println(array.mkString(","))
})
```

##### groupBy

同一个分区的放到一个迭代对象中。结果 tuple 中，第一个元素是 key，后面是 iterator

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 9)
val groupRdd: RDD[(Int, Iterable[Int])] = listRdd.groupBy(i => i % 2)
groupRdd.collect().foreach(println)
--------------------
(0,CompactBuffer(2, 4, 6, 8))
(1,CompactBuffer(1, 3, 5, 7, 9))
```

##### filter

按条件筛选

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 9)
val filterRdd: RDD[Int] = listRdd.filter(_ % 2 == 0)
filterRdd.collect().foreach(println)
```

##### sample

抽样。

参数介绍：withReplacement，是否重复抽样（可重复，泊松抽样；不可重复，伯努利抽样）

fraction，打分？（可重复下，需≥0，代表大概可重复的次数；不可重复下，需[0,1]，代表大概抽取比例）

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 10)
// val sampleRdd: RDD[Int] = listRdd.sample(false, 0.7, 333)
val sampleRdd: RDD[Int] = listRdd.sample(true, 4, 333)
sampleRdd.collect().foreach(println)
```

##### distinct

去重

注意：distinct 计算后，原数据分区会被打乱，是因为中间进行了 shuffle 操作。同时也因为 shuffle 导致必须等待所有分区都计算完成后才能进行下一个操作；而没有 shuffle 操作的算子，执行完一个分区的操作后就可以继续进行下一个操作。

```java
val listRdd: RDD[Int] = sc.makeRDD(List(1, 2, 1, 1, 3, 4, 6, 4, 3))
val disRdd: RDD[Int] = listRdd.distinct()
val disRdd: RDD[Int] = listRdd.distinct(2)  // 设置去重后的分区数
disRdd.collect().foreach(println)
```

##### coalease

缩减分区。实际为合并分区，即将其中某几个分区合并；若要扩大分区，需要添加 shuffle 参数

```java
val listRdd: RDD[Int] = sc.makeRDD(1 to 16, 4)
println("before: ", listRdd.partitions.size)
val coalRdd: RDD[Int] = listRdd.coalesce(3)
println("after: ", coalRdd.partitions.size)
```

##### repartition

对 coalease 的封装，`shuffle = true`

```java
rdd.repartition(2)
```

##### sortBy

排序，可自己设置排序规则

```java
val listRdd: RDD[Int] = sc.makeRDD(List(3, 5, 1, 7, 2))
val sortRdd: RDD[Int] = listRdd.sortBy(x => x)
// val sortRdd: RDD[Int] = listRdd.sortBy(x => x%3)
sortRdd.collect().foreach(println)
```

#### 双 Value 类型

##### union

合并两个 Rdd

```java
val rdd3 = rdd1.union(rdd2)
```

##### subtract

去除相同元素，不同的会保留

```java
val rdd3 = rdd1.subtract(rdd2)
```

##### intersection

求交集后返回

```java 
val rdd3 = rdd1.intersection(rdd2)
```

##### cartesian

笛卡尔积

```java
val rdd3 = rdd1.cartesian(rdd2)
```

##### zip

将两个 rdd 对应元素组合在一起（tuple？key-value？）。两个 rdd 分区数量和元素数量必须都相同；会把分区中的拆成一个一个的元素，组合的元素还在原来的分区里。

```java
val rdd1: RDD[Int] = sc.makeRDD(Array(1, 2, 3, 4), 2)
val rdd2: RDD[String] = sc.makeRDD(Array("a", "b", "c", "d"), 2)
val zipRdd: RDD[(Int, String)] = rdd1.zip(rdd2)
zipRdd.collect().foreach(println)
zipRdd.saveAsTextFile("output")
```

#### Key-Value 类型

##### partitionBy

根据 key 进行重新分区（因此 rdd 需要是 kv 的形式），也可自定义分区类

```java
val arrayRdd: RDD[(Int, String)] = sc.makeRDD(Array((1, "aaa"), (2, "bbb"), (3, "ccc"), (4, "ddd")), 2)
val parRdd: RDD[(Int, String)] = arrayRdd.partitionBy(new org.apache.spark.HashPartitioner(3))
parRdd.saveAsTextFile("output")
```



### Rdd Action 行动算子



### 综合练习



## 二、SparkSQL

Spark SQL是Spark用来处理结构化数据的一个模块，它提供了2个编程抽象：DataFrame和DataSet，并且作为分布式SQL查询引擎的作用。

Rdd → DataFrame → DataSet

- DataFrame：在 Rdd 的基础上，装饰了表结构，让每一个字段包含意义
- DataSet：在 DataFrame 基础上，装饰了读取操作，让数据的读取像操作对象一样简单

```bash
bin/spark-shell
```

### DataFrame

#### 创建

```bash
> val df = spark.read.json("/opt/module/spark-2.3.2-local/mydata/user.json")
> df.show
==========
# user.json
{"name":"123", "age":20}
{"name":"456", "age":20}
{"name":"789", "age":20}
```

#### SQL 风格语法

- 单个 Session 内 View 可见

```scala
> df.createTempView("user")
> spark.sql("select * from user").show()
```

- 创建全局表

```scala
> df.createGlobalTempView("user_g")
> spark.newSession().sql("SELECT * FROM global_temp.user_g").show()
```

#### DSL 风格语法

以对象的方式来操作数据

```scala
> df.select("name").show()
> df.select($"name", $"age" + 1).show()
> df.filter($"age" > 21).show()
> df.groupBy("age").count().show()
```

#### Rdd 转为 DataFrame

```scala
> case class People(name:String, age:Int)
> val rdd1 = sc.makeRDD(List(("zhangsan", 20), ("lisi", 14)))
> val peopleRdd = rdd1.map(t=>{People(t._1, t._2)})
> val df = peopleRdd.toDF
> df.show
```

#### DataFrame 转为 Rdd

注意这里面转换之后，并不会还原成 People 结构，而只是一个 Row 对象。这是因为 DataFrame 本身不存数据的类型

```scala
> df.rdd
```

### DataSet

Dataset是具有强类型的数据集合，需要提供对应的类型信息。

解决 DataFrame 中取数只能通过下标来取的问题（啥意思？？）

#### 创建

```scala
> case class People(name:String, age:Int)
> val caseClassDS = Seq(People("Andy", 21)).toDS()
```

#### Rdd 转换为 DataSet

Rdd + 结构 → DataFrame；DataFram + 类型 → DataSet

Rdd + 结构 + 类型 → DataSet

```scala
> case class Person(name: String, age: Long)
> val mapRdd = rdd.map(t=>{Person(t._1, t._2)})
> val ds = mapRdd.toDS
> ds.show
```

#### DataSet 转换为 Rdd

转换回来仍保留着类型

```scala
> ds.rdd
```

