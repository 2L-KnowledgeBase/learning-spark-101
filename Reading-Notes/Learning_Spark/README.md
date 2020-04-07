# Learning Spark

> [ch ver (*2015*)](https://github.com/2L-knowledgebase/kindle/tree/master/LearningSpark%20%E4%B8%AD%E6%96%87%E7%89%88) & [en ver (*2015*)](https://github.com/2L-knowledgebase/kindle/blob/master/OReilly.Learning.Spark.2015.1.pdf)

## Concepts

- Driver: 每一个 spark 应用程序包含在一个集群上运行各种并行操作的驱动程序

> A Spark driver (aka an application's driver process) is a JVM process that hosts SparkContext for a Spark application

- Spark 应用程序在集群运行时的步骤:
	1. 用户使用 spark-submit 提交一个应用
	2. spark-submit 启动 driver 程序，并调用其中的 main() 方法
	3. driver 程序联系 *集群管理器(Hadoop YARN/Apache Mesos/built-in Standalone)* 请求资源来启动各 executor
	4. *集群管理器* 代表 driver 程序启动各 executor
	5. driver 进程运行整个用户应用，程序中基于 RDD 的变化和动作（有向无环图 DAG) , driver程序以task形式发送到各 executor
	6. task 在 executor 进程中运行来计算和保存结果
	7. 如果 driver 的 main() 方法退出 或者 调用了 SparkContext.stop(), 就会中止 excuter 的运行并释放集群管理器分配的资源

[初始化，构造 SparkConf & SparkContext & SparkSession](../../TechStack/bigdata-spark/spark-configuration.md)

Spark 中的 RDD, 简单来说就是所有对象的一个不可变的分布式集合。每个 RDD 都被分割为多个分区, 这就可以在集群的不同节点上进行计算。RDD 可以包含任何 Python,Java,Scala 对象类型, 包括用户自定义类型。用户可以用两种方式创建 RDD: 
1. 通过加载一个外部数据集(e.g. `SparkContext.textFile()`); 
2. 在驱动程序中分发(`SparkContext.parallelize()`)一个对象集合(如 list 或 set)

RDD 一旦创建好了,可以提供两种不同类型的操作: **变换(transformation)** 和 **动作(action)**
- 变换是从前一个 RDD 构造出一个新的 RDD
- 动作是基于 RDD 来计算某个结果, 并将结果返回给驱动程序 或者 保存结果到一个外部的存储系统(比如 HDFS)

变换和动作不相同是因为 Spark 计算 RDD 的方式。虽然在任何时候你都可以定义一个新的 RDD, **但是 Spark 总是以一种 lazy 的方式计算它们**, 也就是它们被第一次用于动作的时候; 每次你执行个动作 Spark 的 RDD 默认会被重新计算。如果你想在多个动作中重用 RDD, 你可以用 RDD.persist() / RDD.cache() 要求 Spark 对 RDD 持久化。

> `org.apache.spark.storage.StorageLevel`, 如果需要的话还可以通过添加`_2` 到存储级别末尾来复制数据到2台机器

| 存储级别            | 空间占用 | CPU | 在内存 | 在磁盘 | 注释                                                                |
|---------------------|----------|-----|--------|--------|---------------------------------------------------------------------|
| MEMORY\_ONLY         | 高       | 低  | 是     | 否     |                                                                     |
| MEMORY\_ONLY\_SER     | 低       | 高  | 是     | 否     | Store RDD as serialized Java objects (one byte array per partition) |
| MEMORY\_AND\_DISK     | 高       | 低  | 有时   | 有时   | 如果数据太多不能放在内存里,则溢出到磁盘                             |
| MEMORY\_AND\_DISK\_SER | 低       | 高  | 有时   | 有时   | 如果数据太多不能放在内存里,则溢出到磁盘。内存中的数据表现为序列化   |
| DISK\_ONLY           | 低       | 高  | 否     | 是     |                                                                     |

RDD.unpersist()函数用于手动释放缓存

## APIs

> Spark 的 API 很大程度上依靠在驱动程序里传递函数到集群上运行

Java APIs: 传递函数是实现了 org.apache.spark.api.java 包中的 Spark 函数接口的对象。基于函数的返回类型有些不同的接口

| 函数名               | 实现的方法          | 用法                                               |
|----------------------|---------------------|----------------------------------------------------|
| Function<T,R>        | R call(T)           | 一个输入一个输出,用于 map(),filter()之类的操作     |
| Function2<T1,T2,R>   | R call(T1,T2)       | 两个输入一个输出,用于 aggregate(),fold()之类的操作 |
| FlatMapFunction<T,R> | Iterable<R> call(T) | 一个输入零个或多个输出, 用于 flagMap()之类的操作   |

``` java
// 内部匿名类传递函数
RDD<String> errors = lines.filter(new Function<String, Boolean>() {
	public Boolean call(String x) { return x.contains("error"); }
});

// 命名类传递函数
class ContainsError implements Function<String, Boolean>() {
	public Boolean call(String x) { return x.contains("error"); }
}
RDD<String> errors = lines.filter(new ContainsError());

// Java8 的 lambda 表达式传递函数
RDD<String> errors = lines.filter(s -> s.contains("error"));
```

> 特定类型 JavaDoubleRDD 和 JavaPairRDD, 函数的Java接口 

| 函数名                        | 等价函数\*<A,B,...>                 | 用法                                |
|-------------------------------|------------------------------------|-------------------------------------|
| DoubleFlatMapFunction<T>      | Function<T, Iterable<Double>>      | 从 flatMapToDouble()得到 DoubleRDD  |
| DoubleFunction<T>             | Function<T, double>                | 从 mapToDouble()得到 DoubleRDD      |
| PairFlatMapFunction <T, K, V> | Function<T, Iterable<Tuple2<K,V>>> | 从 flatMapToPair()得到 PairRDD<K,V> |
| PairFunction<T, K, V>         | Function<T, Tuple2<K,V>>           | 从 mapToPair()得到 PairRDD<K,V>     |


### RDD(resilient distributed dataset, 弹性分布式数据集) APIs

**有些函数只对某种类型的RDD可用,比如mean()和variance()对数值类型的RDD可用,而join()对键值对类型的RDD可用**

#### Transformation

> 对一个包含{1, 2, 3, 3}的RDD进行基本的变换

| 函数名                                   | 目的                                                         | 示例                        | 结果                  |
|------------------------------------------|--------------------------------------------------------------|-----------------------------|-----------------------|
| map()                                    | 应用函数到 RDD 中的每个元素,并返回一个结果RDD                | rdd.map( x => x + 1 )       | {2, 3, 4, 4}          |
| flatMap()                                | 应用函数到 RDD 中的每个元素,并返回一个返回的迭代器内容的 RDD | rdd.flatMap( x => x.to(3) ) | {1, 2, 3, 2, 3, 3, 3} |
| filter()                                 | 返回由仅通过传入filter()的条件的元素组成的 RDD               | rdd.filter( x => x != 1 )   | {2, 3, 3}             |
| distinct()                               | 去重                                                         | rdd.distinct()              | {1, 2, 3}             |
| sample(withReplacement,fraction, [seed]) | RDD 采样数据（有放回/无放回 抽样）                           | rdd.sample(false, 0.5)      | 随机抽样              |

> 对包含{1, 2, 3}和{3, 4, 5}的两个RDD进行变换 

| 函数名         | 目的                                          | 示例                    | 结果                         |
|----------------|-----------------------------------------------|-------------------------|------------------------------|
| union()        | 生成一个包含两个 RDD中所有元素的 RDD          | rdd.union(other)        | {1, 2, 3, 3, 4, 5}           |
| intersection() | 生成两个 RDD 中都有的元素组成的 RDD           | rdd.intersection(other) | {3}                          |
| subtract()     | 从一个 RDD 中去掉另一个 RDD 中 存 在 的 元 素 | rdd.subtract(other)     | {1, 2}                       |
| cartesian()    | 生成两个 RDD 的笛卡尔积的 RDD                 | rdd.cartesian(other)    | {(1, 3), (1, 4), ..., (3,5)} |

> 单个pair RDD的变换, 以{(1,2), (3,4), (3,6)}为例, [e.g. JavaPairRDD](https://spark.apache.org/docs/latest/api/java/index.html)

| 函数名                                                                | 目的                                                                                | 示例                               | 结果                                           |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------|------------------------------------|------------------------------------------------|
| reduceByKey(func)                                                     | 按相同的键合并                                                                      | rdd.reduceByKey( (x, y) => x + y ) | {(1,2), (3,10)}                                |
| groupByKey()                                                          | 按相同的键分组                                                                      | rdd.groupByKey()                   | {(1,[2]),(3,[4,6])}                            |
| combineByKey(createCombiner, mergeValue, mergeCombiners, partitioner) | 按相同的键合并, 返回不同的结果类型(区别于reduceByKey)                               |                                    |                                                |
| mapValues(func)                                                       | 应用函数到 pairRDD 的每个值, 但是不改变键                                           | rdd.mapValues(x => x+1)            | {(1,3), (3, 5), (3,7)}                         |
| flatMapValues(func)                                                   | 应用一个返回pair RDD 中每个值的迭代器的函数, 并对每个返回的元素以原来的键生成键值对 | rdd.flatMapValues( x => (x to 5) ) | {(1, 2), (1, 3),(1, 4), (1, 5), (3,4), (3, 5)} |
| keys()                                                                | 只返回 RDD 中的所有键                                                               | rdd.keys()                         | {1, 3, 3}                                      |
| values()                                                              | 只返回 RDD 中的所有值                                                               | rdd.values()                       | {2, 4, 6}                                      |
| sortByKey()                                                           | 返回按键排序的RDD                                                                   | rdd.sortByKey()                    | {(1, 2), (3, 4), (3, 6)}                       |


> 两个pair RDD的变换, 以(rdd={(1,2), (3,4), (3,6)},other={(3,9)}) 为例

| 函数名        | 目的                                               | 示例                     | 结果                                           |
|---------------|----------------------------------------------------|--------------------------|------------------------------------------------|
| subtractByKey | 去除另一个RDD中存在键的元素                        | rdd.subtractByKey(other) | {(1, 2)}                                       |
| join          | 两个RDD执行内连接                                  | rdd.join(other)          | {(3, (4, 9)), (3, (6,9)}                       |
| rightOutJoin  | 两个RDD执行连接操作, 但是other RDD中的key 必须存在 | rdd.leftOutJoin(other)   | {(1,(2,None)),(3,(4,Some(9))),(3,(6,Some(9)))  |
| leftOutJoin   | 两个RDD执 行连接操作,但是第一个RDD中的key必须存在  | rdd.leftOutJoin(other)   | {(1,(2,None)),(3,(4,Some(9))),(3,(6,Some(9)))} |
| cogroup       | 对两个RDD的数据共享相同的键分组                    | rdd.cogroup(other)       | {(1, ([2], [])),(3,([4, 6], [9]))}             |

> (预先)分区相关, `repartition()` / `coalesce()` / `paritionBy()`*only pairRDD*`

#### Action

> 对包含{1, 2, 3, 3}的RDD执行动作

| 函数名                              | 目的                                 | 示例                                                                                    | 结果                     |
|-------------------------------------|--------------------------------------|-----------------------------------------------------------------------------------------|--------------------------|
| collect()                           | 返回RDD中的所有元素                | rdd.collect( )                                                                          | {1, 2, 3, 3, 4, 5}       |
| count()                             | 返回RDD中元素个数                  | rdd.count()                                                                             | 4                        |
| countByValue()                      | RDD中每个元素出现的次数             | rdd.countByValue()                                                                      | {(1, 1), (2, 1), (3, 2)} |
| take(num)                           | 返回RDD中的num                   | rdd.take(2)                                                                             | {1, 2}                   |
| top(num)                            | 返回RDD中前num个元素           | rdd.top(2)                                                                              | {3, 3}                   |
| [reduce(func)](./rdd-apis-example.md#reduce)                        | 并行合并 RDD 中的元素                | rdd.reduce((x,y) => x+y)                                                                | 9                        |
| fold(func)                          | 和reduce()一样,但是提供了一个初值   | rdd.fold(0)((x,y)=>x+y)                                                                 | 9                        |
| [aggregate(zeroValue)(seqOp, combOp)](./rdd-apis-example.md#aggregate) | 类似 reduce(),但是用于返回不同的类型 | rdd.aggregate((0,0)((x, y) => (x._1 + y, x._2 +1),(x, y) => (x._1 + y._1, x._2 + y._2)) | (9, 4)                   |
| foreach(func)                       | 对RDD中的每个元素应用函数func     | rdd.foreach(func)                                                                       | Noting                   |


### DataSet(Dataframe) APIs

> RDD vs DataFrame vs DataSet, [refer to @migrate from spark1.x to spark2.x](../../TechStack/bigdata-spark/spark1_to_spark2.md)

### 高级特性

- 累计器, `SparkContext.accumulator`
- 广播变量，`org.apache.spark.broadcast.Broadcast[T]`
- 基于分区工作, `mapParitions()` & `foreachPartitions()` 
- 管道，`pipe()`
- 数值类型RDD，`DoubleRDD`
