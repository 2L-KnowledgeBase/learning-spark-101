## Migrate from spark1.x to spark2.x

### SparkSession

> sparkSession 可以视为 SqlContext 和 HiveContext 以及 StreamingContext的结合体，**这些Context的API都可以通过sparkSession使用**

``` java
import org.apache.spark.sql.SparkSession;

SparkSession spark = SparkSession.builder()
// 本地2个线程; SparkonYARN 不需要指定master
.master("local[2]")
.appName(appNm)
// 使用enableHiveSupport就能够支持hive，相当于hiveContext
.enableHiveSupport()
// <200mb, broadcast join
.config("spark.sql.autoBroadcastJoinThreshold","104875600")
.config("spark.sql.shuffle.partitions","500")
.config("spark.sql.qubole.split.computation","true")
.getOrCreate();
```

### RDD vs DataFrame vs DataSet

> DataFrame & DataSet are both built on top of RDD

1. RDD – The RDD APIs have been on Spark since the 1.0 release.
2. DataFrame – Spark introduced DataFrame in Spark 1.3 release.
3. DataSet – Spark introduced Dataset in Spark 1.6 release (alpha).
4. DataFrame and DataSet are unified in Spark 2.0 release. *Conceptually, consider DataFrame as an alias for a collection of generic objects Dataset[Row], where a Row is a generic untyped JVM object.*

**Converted to each others with `toDS()` & `toDF()` & `toJavaRDD()` methods**

``` java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

# https://spark.apache.org/docs/2.3.0/sql-programming-guide.html#creating-dataframes
# In the Scala API, DataFrame is simply a type alias of Dataset[Row]. 
# While, in Java API, users need to use Dataset<Row> to represent a DataFrame.
Dataset<Row> df = spark.read()
                    .option("header","true")
                    .csv("src/main/resources/test.csv");
```                    

RDD & Dataset are both **compile-time type safety**.

DataFrame works only on structured and semi-structured data. It organizes the data in the named column. DataFrames allow the Spark to manage schema.
- [DataFrameReader API @spark2.3.1](https://spark.apache.org/docs/2.3.1/api/java/org/apache/spark/sql/DataFrameReader.html)
- [DataFrameWriter API @spark2.3.1](https://spark.apache.org/docs/2.3.1/api/java/org/apache/spark/sql/DataFrameWriter.html)

> [Master in SparkSQL DataSource API — Managing Datasets in External Data Sources](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/spark-sql-datasource-api.html)


> references:

1. [将代码从 spark 1.x 移植到 spark 2.x](https://www.jianshu.com/p/fb9722809165)
2. [@CSDN: CDH集群 Spark1.6 升级到 Spark2.2 全纪录](https://blog.csdn.net/Abysscarry/article/details/79550746)
3. [@CSDN: CDH5.11 离线安装或者升级 spark2.x 详细步骤](https://blog.csdn.net/u010936936/article/details/73650417)
4. [Apache Spark RDD vs DataFrame vs DataSet](https://data-flair.training/blogs/apache-spark-rdd-vs-dataframe-vs-dataset/)
5. [@Databrciks: Three Apache Spark APIs: RDDs vs DataFrames and Datasets, When to use them and why?](https://databricks.com/blog/2016/07/14/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets.html)
