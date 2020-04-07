#### Install Spark

Download spark via https://spark.apache.org/downloads.html .

Enusre you have JDK(>=1.8) installed yet first (`java -version`). Since Spark uses Hadoop’s client libraries for HDFS and YARN, you can choose any of ways below: 
- download a pr-built spark(aka binary version) with a specified hadoop version, e.g. Chose a package type "Pre-built for Apache Hadoop 2.7 and later"
- download a pr-built spark only (choose "Pre-built with user-provided Apache Hadoop") and setup spark CLASSPATH like below.
	```
	# download any hadoop binary version (e.g. 2.8.2) and setup Pseudo-Distributed 单机伪分布

	# setup spark CLASSPATH
	vi conf/spark-env.sh
	export SPARK_DIST_CLASSPATH=$(hadoop classpath)	
	```
- download spark source code (or clone from github), then build it with specified hadoop dependency

**For version sepecific, YOU SHOULD KNOW** :
1. Spark runs on Java 8, Python 2.7+/3.4+ and R 3.1+. For the Scala API, Spark 2.4.4 uses Scala 2.12. You will need to use a compatible Scala version (2.12.x).
2. Support for Java 7, Python 2.6 and old Hadoop versions before 2.6.5 were removed as of Spark 2.2.0.
3. Support for Scala 2.10 was removed as of 2.3.0.
4. Support for Scala 2.11 is deprecated as of Spark 2.4.1 and will be removed in Spark 3.0.

#### Run a Example Locally

``` bash
$ cd spark-2.4.4-bin-hadoop2.7

$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
                     --master local[2] \
                     examples/jars/spark-examples_2.11-2.4.4.jar 10

# --class          指定 spark app 的入口类, 必须包含 main 方法
# --master         指定 master URL, local[2] 表示 本地执行 & 2个worker线程
# examples/xxx.jar 指定 含有入口类的 jar包路径
# 10               入口类 main 方法接受的参数 String[] args

# 这是一个计算圆周率的示例
```

> According to different deploy modes of spark application.

#### Demo of A Spark Standone Cluster

> One master & one worker run on same machine in two processes.

``` bash
$ sbin/start-all.sh
starting org.apache.spark.deploy.master.Master, logging to /xxx/spark-2.4.4-bin-hadoop2.7/logs/spark-geluo-org.apache.spark.deploy.master.xxx.out
localhost: starting org.apache.spark.deploy.worker.Worker, logging to /xxx/spark-2.4.4-bin-hadoop2.7/logs/spark-geluo-org.apache.spark.deploy.worker.Worker-1-xxx.out

# Access web UI of Spark Master at http://localhost:8080
# You could replace "localhost" with your actual hostname/machine name if needed!

$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
                     --master spark://localhost:7077 \
                     examples/jars/spark-examples_2.11-2.4.4.jar 10

# --deploy-mode: default is client
```

> `spark://localhost:7077` is the **default master url(traditional RPC gateway)** of Spark Master.

![spark_master_web_ui](https://user-images.githubusercontent.com/4146503/76157960-5681a580-614b-11ea-9227-732af34decc3.png)

