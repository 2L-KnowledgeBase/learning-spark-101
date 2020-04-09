More details refer to https://spark.apache.org/docs/latest/submitting-applications.html .


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