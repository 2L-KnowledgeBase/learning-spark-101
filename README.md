# Learning Spark 101

> http://spark.apache.org

#### What is Spark ?

Spark is a `___` ? Currently (these definitions are changable in developer community and updated overtime) we said ...

*Spark is a unified analytics engine for large-scale data processing.*

> 关于 *是什么的定义(what-is)* 时常随着 开源项目本身的发展 以及 PMC Member们的协商 而进行调整 (下一个Major Releasae), 简单来说, Spark是一个分布式计算框架(引擎), 很多开发者常喜欢用它和MapReduce做比较.

- [Why Spark is better than Hadoop? What is the difference between them?](mapreduce-vs-spark.md)

It provides high-level [APIs in Scala/Java/Python and R](https://spark.apache.org/docs/2.4.3/api.html), and an optimized engine that supports general computation graphs(**DAG, Directed Acyclic Graph**) for data analysis. 

> 高阶 API 又可以划分为 RDD / Dataset (Dataframe, Spark>=2.0之后 Dataframe API 和 Dataset 进行了合并) 2大类, 后者是社区参考著名的 Python 包 Pandas 的 API 设计风格(更利于分析师使用) 二次封装 RDD API 实现 

It also supports a rich set of higher-level tools including Spark SQL for SQL and DataFrames, MLlib for machine learning, GraphX for graph processing, and Structured Streaming for stream processing.

> 除了可以直接使用上面提到的 [RDD](RDD/README.md)/Dataset APIs 编写 SparkApplicaiton 之外. Spark 还提供如下针对特定场景的内嵌库 (built-in library).

<img width="300" alt="spark stack" src="http://spark.apache.org/images/spark-stack.png">

### - [Structured Dataset Query & SQL](SparkSQL/README.md)

> SparkSQL在构建现代化数仓中相当流行, *现代化数仓*这一中文概念，请参考[eBay的俞育才老师在18年QCON上的sharing](Reading-Notes/eBay-俞育才-构建现代化数据仓库.v4.pdf)，由于MPP(e.g. Teradata/Netezza/Greenplum/..)本身高昂的成本, 越来越多的公司开始将离线分析从传统数仓迁移到现代化数仓(HDFS + Hive + SparkSQL).

> 关于 eBay DSS(Data Services and Solutions) Team 完成迁移的sharing 从 "Spark& AI Summit Europe - 2018/10" 开始到国内的QCon/Spark&AI Meetup 等等.. 真的有很多.. 以下为Databricks Youtube账号提供的录制(自备梯子)
1. [Moving eBay’s Data Warehouse Over to Apache Spark (Kimberly Curtis & Brian Knauss)](https://www.youtube.com/watch?v=d410R_H9FX0)
2. [Analytical DBMS to Apache Spark Auto Migration Framework with Edward Zhang and Lipeng Zhu (eBay)](https://www.youtube.com/watch?v=dahIDF0SKPw)
3. [Deep Dive of ADBMS Migration to Apache Spark—Use Cases Sharing - Keith Sun eBay](https://www.youtube.com/watch?v=i-L2wtN9tyg)
4. [Experience Of Optimizing Spark SQL When Migrating from MPP Database - Yucai Yu and Yuming Wang eBay](https://www.youtube.com/watch?v=BcenJqszr6g)

### - [Structured Streaming](SparkStreming/README.md)

> SparkStreaming由于Spark2.0后Dataset API的统一, 使得SparkStreaming使用Spark SQL APIs

### - [GraphX](GraphX/README.md)

### - [MLlib](MLlib/README.md)

#### [How to setup Spark?](installation.md)

#### [Submit Spark Application](submit.md)

#### [Spark Configuration](configuration.md)

### Awesome Study Resources

Spark lastest version doc: http://spark.apache.org/docs/latest/. For historical versions, find [here](http://spark.apache.org/documentation.html).

1. 《Learning Spark》 ([En](https://github.com/KnowledgeBase-ForAnEngineer/kindle/blob/master/OReilly.Learning.Spark.2015.1.pdf) / [译版2-9章](https://github.com/2L-knowledgebase/kindle/tree/master/LearningSpark%20%E4%B8%AD%E6%96%87%E7%89%88)) [@Notes](ReadingNotes/Learning_Spark)
2. [Mastering Spark SQL](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/)
3. [Mastering Apache Spark](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/)
4. [Advanced Apache Spark Training - Sameer Farooqui (Databricks)](https://www.youtube.com/watch?v=7ooZ4S7Ay6Y&feature=youtu.be)

> Spark usage scenarios @PayPal (publicly exposed only)
1. [SCaaS: Spark Compute as a Service at Paypal - Prabhu Kasinathan](https://www.youtube.com/watch?v=Oqq3m4RP2tE)
2. [Merchant Churn Prediction Using SparkML at PayPal (Chetan Nadgire and Aniket Kulkarni)](https://www.youtube.com/watch?v=v6EF1_AVvKU)
3. [Graph Representation Learning to Prevent Payment Collusion Fraud (Venkatesh Ramanathan)](https://www.youtube.com/watch?v=eq_rpur1eNM)
4. [PayPal Merchant ecosystem using Spark, Hive, Druid, HBase & Elasticsearch](Reading-Notes/PayPal%20Merchant%20ecosystem%20using%20Spark%2C%20Hive%2C%20Druid%2C%20HBase%20%26%20Elasticsearch.pdf)

### Read Source Code & Contribute to the Community

1. [Shared Analysis Track](Source-Code-Analysis/README.md)
2. [Known issues & Contribution](Contribution/README.md)
