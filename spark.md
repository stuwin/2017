## Learning Spark

start: 2017-04-22 end:

Spark是快速通用的集群计算平台

通用：容纳了其他分布式系统拥有的功能，如批处理、迭代式计算、交互查询和流处理等。

1.Spark介绍与生态
-----------------

Spark是紧密集成的。

组件和生态: Spark Core, Spark SQL, Spark Streaming, Mlib, Graphx, Cluster Manager

与Hadoop的对比：时效性高、机器学习

2.Spark下载安装和Shell
-----------------

Dependency:

Spark 1.6.2 - Scala 2.10
Spark 2.0.0 - Scala 2.11

Spark Python Shell

Spark Scala Shell

修改shell的日志输出级别 log4j.properties

3.Spark开发环境搭建
-----------------

My Environment:

```
Java 1.8
Spark 1.6.2 (hadoop 2.6)
Scala 2.10.5
SBT 0.13.8
```

4.Spark RDD Transformation
----------------------

参考资料：

知乎讨论（flatMap 与 Map）https://www.zhihu.com/question/34548588?sort=created

Transformation 简介

Transformation 属于 Spark RDD 的常用基本操作中的一类，可以理解为其他语言中的函数或方法，其作用是：基于一个RDD，根据一定的规则，构建一个新的RDD。

BASE_RDD_OBJ ===Transformation===> NEW_RDD_OBJ

Input ===Method===> Output

常用Transformation介绍

**map**

名称： map
作用：对RDD中的每个元素，运行指定规则进行计算，计算结果为元素，并返回结果集。
实例：

```
val lines = sc.parallelize(Array("hello","spark","hello","world","!"))
val lines_map = lines.map(word=>(word,1))
lines_map.foreach(println)
```

# 返回结果为

```
(hello,1)
(spark,1)
(hello,1)
(world,1)
(!,1)
```

**filter**

名称： filter
作用：过滤RDD中的元素，返回符合指定规则的元素集。
实例：

```
val lines = sc.parallelize(Array("hello","spark","hello","world","!"))
val lines_filter = lines.filter(word=>word.contains("r"))
lines_filter.foreach(println)
```

# 返回结果为 
spark
world

**flatMap**

名称： flatMap
作用：对RDD中的每个元素，运行指定规则进行计算，计算结果为集合，并将每个元素的结果集合并为一个集合后输出为新的RDD。
实例：

```
# =================== hello.txt ===========================
# Hello world
# Hello spark
# some world
```

```
val input = sc.textFile("hello.txt")
val input_flatmap = input.flatMap(line=>line.split(" "))
input_flatmap.foreach(println)
```

# 返回结果为

```
Hello
world
Hello
spark
some
world
```

**distinct**

名称： distinct
作用： 将RDD中重复的元素保留一份，即去重。
实例：

```
val rdd1 = sc.parallelize(Array("coffee","coffee","panda","monkey","tea"))
val rdd_distinct = rdd1.distinct()
rdd_distinct.foreach(println)
```

# 返回结果为

``` 
monkey
coffee
panda
tea
```

**union**

名称： union
作用：将RDD与参数中的RDD进行元素合并。
实例：

```
val rdd1 = sc.parallelize(Array("coffee","coffee","panda","monkey","tea"))
val rdd2 = sc.parallelize(Array("coffee","monkey","kitty"))
val rdd_union = rdd1.union(rdd2)
rdd_union.foreach(println)
```

# 返回结果为

``` 
coffee
coffee
panda
monkey
tea
coffee
monkey
kitty
```

intersection

名称： intersection
作用： 与参数中的RDD进行比较，进行相交运算，返回两个RDD中共有的元素。
实例：

```
val rdd1 = sc.parallelize(Array("coffee","coffee","panda","monkey","tea"))
val rdd2 = sc.parallelize(Array("coffee","monkey","kitty"))
val rdd_intersection = rdd1.intersection(rdd2)
rdd_intersection.foreach(println)
```

# 返回结果为

``` 
monkey
coffee
```

**subtract**

名称： subtract
作用： 从RDD中剔除参数RDD中所包含的元素，并返回。
实例：

```
val rdd1 = sc.parallelize(Array("coffee","coffee","panda","monkey","tea"))
val rdd2 = sc.parallelize(Array("coffee","monkey","kitty"))
val rdd_sub = rdd1.subtract(rdd2)
rdd_sub.foreach(println)
```

# 返回结果为

``` 
tea
panda 
```

参考文章 http://blog.csdn.net/danielpei1222/article/details/65953931

5.Spark RDD Action
------------------

**reduce**

接收一个函数，作用在RDD两个类型相同的元素上，返回新元素；可以实现RDD中元素的累加，计数和其他类型的聚集操作。

**collect**
遍历RDD，向Driver Pragram返回RDD的内容

**take(n)**
返回RDD的n个元素（同时尝试返回最少的partitions），返回结果是无序的

**top(n)**
排序（根据RDD中默认比较器）

**foreach**
计算RDD中的每个元素，但不返回到本地，配合print友好打印数据

6.RDDs的特性
-----------

**血统关系图**

Spark维护着RDDs之间的依赖关系和创建关系，叫做血统关系图

Spark使用血统关系图来计算每个RDD的需求和恢复丢失的数据

**延迟计算（Lazy Evaluation**

Spark对RDDs的计算是它们第一次使用Action操作，这种方式在处理大数据的时候特别有用，可以减少数据的传输。

Spark内部记录metadata表明Transformation已经被响应。

加载数据也是延迟计算，数据只有在必要的时候，才会被加载进去。

**RDD.persist()**

重用RDD（默认在RDDs上进行Action操作时，会重新计算RDDs）
unpersist()：从缓存中移除

RDD.persist()参数

|    级别                | 空间占用 | CPU消耗 | 是否在内存中 | 是否在硬盘上 |
| ---------------------- | ------- | ------ | ---------- | ---------- |
|    MEMORY_ONLY         | High    | Low    | Y          |  N         |
|    MEMORY_ONLY_SEQ     | Low     | High   | Y          |  N         |
|    DISK_ONLY           | Low     | High   | N          |  Y         |
|    MEMORY_AND_DISK     | High    | Medium | Some       | Some       |
|    MEMORY_AND_DISK_SEQ | Low     | High   | Some       | Some       |


