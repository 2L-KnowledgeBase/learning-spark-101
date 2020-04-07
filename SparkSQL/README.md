## Get Started ![](http://spark.apache.org/images/spark-logo-trademark.png)

> [spark 2.2.0 中文指南](http://spark.apachecn.org/docs/cn/2.2.0/index.html)

1. download apache spark, via http://spark.apache.org/downloads.html
	- Spark 可以通过 Hadoop client 库使用 HDFS 和 YARN。下载一个预编译主流 Hadoop 版本比较麻烦。用户可以下载一个编译好的 Hadoop 版本，并且可以 通过设置 Spark 的 classpath 来与任何的 Hadoop 版本一起运行 Spark
	```
	# download hadoop 2.7.2 and setup Pseudo-Distributed

	# set spark CLASSPATH, http://spark.apachecn.org/docs/cn/2.2.0/hadoop-provided.html
	vi conf/spark-env.sh
	export SPARK_DIST_CLASSPATH=$(hadoop classpath)
	```
	- Spark 可运行在 Java 8+，Python 2.7+/3.4+ 和 R 3.1+ 的环境上。针对 Scala API，Spark 2.2.0 使用了 Scala 2.11。您将需要去使用一个可兼容的 Scala 版本（2.11.x）。
	- 请注意，从 Spark 2.2.0 起，对 Java 7，Python 2.6 和旧的 Hadoop 2.6.5 之前版本的支持均已被删除。
2. interactive shell (`spark-shell` / `pyspark`)
	- 2.x: `bin/pyspark` or `bin/pyspark --master local[2]`, SparkSession available as 'spark'
	- 1.x: `bin/pyspark` or `bin/pyspark --master local[2]`, SparkContext available as sc, HiveContext available as sqlContext.
3. spark-sql (similiar to hive-cli / beeline)
4. `spark submit` to submit a spark app
	```
	./bin/spark-submit --class org.apache.spark.examples.SparkPi \
	--master yarn-cluster \
	--num-executors 3 \
	--driver-memory 4g \
	--executor-memory 2g \
	--executor-cores 1 \
	lib/spark-examples*.jar \
	10
	
	# maven build and run
	mvn clean && mvn compile && mvn package
	spark-submit --master \
	--class com.package_xx.WordCount \
	./target/xxx.jar \
	./README.md ./wordcounts
	
	# CDH running spark application on YARN
	# https://www.cloudera.com/documentation/enterprise/5-11-x/topics/cdh_ig_running_spark_on_yarn.html
	spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster $SPARK_HOME/lib/spark-examples.jar 10
	
	18/08/27 11:26:16 INFO yarn.Client: Application report for application_1528103907147_4900 (state: ACCEPTED)
	
	yarn logs -applicationId application_1528103907147_4900
	```

### Indexs

- [RDD 编程](./rdd.md)
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

### references

)

### Q & A

1. [How to load local file in sc.textFile, instead of HDFS?](https://stackoverflow.com/questions/27299923/how-to-load-local-file-in-sc-textfile-instead-of-hdfs) `lines = sc.textFile("file:///home/XXX/spark-1.6.3-bin-hadoop2.6/README.md")`
2. [http://sqlandhadoop.com/how-to-implement-recursive-queries-in-spark/](http://sqlandhadoop.com/how-to-implement-recursive-queries-in-spark/)
