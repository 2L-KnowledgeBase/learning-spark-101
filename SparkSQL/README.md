# Structured Dataset Query & SQL

### DataFrameReader & [DataFrameWriter](https://spark.apache.org/docs/2.4.3/api/java/index.html)

<Learing Spark, Chapter3>:" Users create RDDs in two ways: by loading an external dataset, or by distributing a collection of objects (e.g., a list or set) in their driver program (e.g. `JavaRDD<String> lines = sc.parallelize(Arrays.asList("pandas", "i like pandas"));`).

As "by loading an external dataset" mentioned here, we usually leverage [DataFrameReader APIs](https://spark.apache.org/docs/2.4.3/api/java/index.html) to do this work. **Don't feel confused with this name since it's just a legacy name and you can also treat it as "DatasetReader"**.

Natively, DataFrameReader support reading different types of files (csv/parquet/orc/text/json) from both local or HDFS (`file://` vs `hdfs://`) and jdbc way to connect relational databases. There are also some thrid party Dataset Reader/Writer extensions, for example:
1. MongoDB: https://docs.mongodb.com/spark-connector/master/java/datasets-and-sql/
2. ElasticSearch: https://www.elastic.co/guide/en/elasticsearch/hadoop/6.5/spark.html#spark-sql-write
3. Hbase:
	- CDH: https://github.com/cloudera-labs/SparkOnHBase
	- Hortonworks: [Spark HBase Connector: Feature Rich and Efficient Access to HBase Through Spark SQL](https://www.youtube.com/watch?v=kU45zMl7TqA)

Similiar to create RDDs, you can retire a RDD in two ways:
- write out to external system (e.g. it’s common to write data out to a distributed storage system such as HDFS)
- collect into driver (e.g. `take()`, `collect()`), **In most cases RDDs can’t just be collect()ed to the driver (as well as `parallelize()`) because they are too large.**

> [What's the different between Dataset, Dataframe and RDD?](./dataset-dataframe-rdd.md)

### Indexs

- [Dataframe API, since spark1.3](./dataframe.md)
- [spark known bugs](./spark-bugs.md)
- [spark configuration](./spark-configuration.md)
- [migrate from spark1.x to spark2.x](spark1_to_spark2.md)
- [如何编译spark源码?](./build-source.md)
- [shuffle](./shuffle.md)
- [spark shell](./spark-shell.md)

> sparkSQL

- [sparkSQL .sql() supported SQL syntax](./spark-sql-syntax.md)
- [sparkSQL general errors](./spark-sql-errors.md)
- [default file format - parquet](./parquet.md)
- [(hive-support) what's bucket table & why use it?](./bucket-table.md)
- [sparkSQL adaptive execution](./spark-with-AE.md)

### SparkSQL Limits 

1. [Recursive Query](http://sqlandhadoop.com/how-to-implement-recursive-queries-in-spark/)

### Legacy of 1.x

1. `Dataframe` APIs (>=1.3, <2.0)
2. Migrate from 1.x to 2.x