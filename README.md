## Learning Spark 101

> http://spark.apache.org

### What is Spark ?

Spark is a `___` ? Currently (these definitions are changable in developer community and updated overtime) we said ...

*Spark is a unified analytics engine for large-scale data processing.*

> 关于 *是什么的定义(what-is)* 时常随着 开源项目本身的发展 以及 PMC Member们的协商 而进行调整 (下一个Major Releasae), 简单来说, Spark是一个分布式计算框架(引擎), 很多开发者常喜欢用它和MapReduce做比较.

**#Popular Topic#**: [*Why Spark is better than Hadoop? What is the difference between them?*](./mapreduce-vs-spark.md)

It provides high-level [APIs in Scala/Java/Python and R](https://spark.apache.org/docs/2.4.3/api.html), and an optimized engine that supports general computation graphs(**DAG, Directed Acyclic Graph**) for data analysis. 

> 高阶 API 又可以划分为 RDD / Dataset (Dataframe, Spark>=2.0之后 Dataframe API 和 Dataset 进行了合并) 2大类, 后者是社区参考著名的 Python 包 Pandas 的 API 设计风格(更利于分析师使用) 二次封装 RDD API 实现 

It also supports a rich set of higher-level tools including Spark SQL for SQL and DataFrames, MLlib for machine learning, GraphX for graph processing, and Structured Streaming for stream processing.

> 除了可以直接使用上面提到的 [RDD](./RDD/README.md)/Dataset APIs 编写 SparkApplicaiton 之外. Spark 还提供如下针对特定场景的内嵌库 (built-in library).

![](http://spark.apache.org/images/spark-stack.png)

#### [Structured Dataset Query & SQL](./SparkSQL/README.md)

> *SparkSQL在构建现代化数仓中相当流行*

#### [Structured Streaming](./SparkStreming/README.md)

> *SparkStreaming由于Spark2.0后Dataset API的统一, 使得SparkStreaming可以受利于Spark SQL*

#### [GraphX](./GraphX/README.md)

> *图计算*

#### [MLlib](./MLlib/README.md)

> *机器学习*

### Awesome Study Resources

Spark lastest version doc: http://spark.apache.org/docs/latest/. For historical versions, find [here](http://spark.apache.org/documentation.html).

1. 《Learning Spark》 ([En](https://github.com/KnowledgeBase-ForAnEngineer/kindle/blob/master/OReilly.Learning.Spark.2015.1.pdf) / [译版2-9章](https://github.com/2L-knowledgebase/kindle/tree/master/LearningSpark%20%E4%B8%AD%E6%96%87%E7%89%88)) [@读书笔记](./eBooks/Learning_Spark)
2. [Mastering Spark SQL](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/)
3. [Mastering Apache Spark](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/)
4. [Advanced Apache Spark Training - Sameer Farooqui (Databricks)](https://www.youtube.com/watch?v=7ooZ4S7Ay6Y&feature=youtu.be)


### [How to setup Spark?](./installation.md)

### [Submit Spark Application](./submit.md)

### [Spark Configuration](./configuration.md)