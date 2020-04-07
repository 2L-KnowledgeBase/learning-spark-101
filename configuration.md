## Spark Configuration

> https://spark.apache.org/docs/2.4.1/configuration.html, change the version-num in url as u need (e.g. 2.4.1).

### Spark properties

> Spark leverage tranditional Java `.properties` file style `key.aaa.bbb=value`

Spark has **lots properties** to be configured by contructing a `SparkConf` object through Java system properties:
https://github.com/apache/spark/blob/branch-2.4/core/src/main/scala/org/apache/spark/deploy/SparkApplication.scala#L36

``` scala
// 利用 JVM System Property 传递配置项 e.g spark.master = spark://localhost:7077
    val sysProps = conf.getAll.toMap
    sysProps.foreach { case (k, v) =>
      sys.props(k) = v
    }
```

In your **driver program**, with `SparkConf`, you can configure some of the common properties (e.g. master URL and application name), as well as key-value pairs through the set() method.

``` scala
val conf = new SparkConf()
            	.setMaster("local[2]")
            	.setAppName("WordCount")
val sc = new SparkContext(conf)

// or
conf.set("spark.master", "local[2]")
conf.set("spark.app.name", "WordCount")

// or
val sparkSession = SparkSession.builder
                   	.master("master")
                   	.appName("appName")
                   	.getOrCreate()

// or
val sparkSession = SparkSession.builder
                    .config("spark.master", "local[2]")
                    .config("spark.app.name", "appName")
                    .getOrCreate()

// or
val sparkSession = SparkSession.builder.config(conf=SparkConf()).getOrCreate()
```

**Otherwise a empty SparkConf in driver is also ok**, `val sc = new SparkContext(new SparkConf())`. Then, you can supply configuration values at runtime (**avoid hard-coding certain configurations in a SparkConf**)

``` bash
./bin/spark-submit --name "My app" \
--master local[4] \
--conf spark.eventLog.enabled=false \
--conf "spark.executor.extraJavaOptions=-XX:+PrintGCDetails -XX:+PrintGCTimeStamps" myApp.jar
```

> `spark-shell` and `spark-submit` support **two ways** to load configurations dynamically. 

The first is command line options, such as --master, as shown above. 
- `spark-submit` can accept any Spark property using the --conf flag
- special flags (e.g. `--master`), running `./bin/spark-submit --help` will show the entire list

All property keys are started with `spark.aaa.bbb`, e.g. Spark UI related properties are started with `spark.ui.aaa.bbb=value`.

**However, as mentioned above, there are so many properties that some of them are not documented on offical doc site. For example, you can find more Spark SQL configurations [here in <Mastering Spark SQL>](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-properties.html).**

[Dynamic Allocation (of Executors)](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-dynamic-allocation.html) & [ExternalShuffleService](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-ExternalShuffleService.html)
```
spark.dynamicAllocation.enabled=true;
spark.dynamicAllocation.minExecutors=2;
spark.dynamicAllocation.maxExecutors=100;
spark.shuffle.service.enabled=true;

# Dynamic allocation is enabled using spark.dynamicAllocation.enabled setting. When enabled, it is assumed that the External Shuffle Service is also used (it is not by default as controlled by spark.shuffle.service.enabled property).

# AE: For users who enabled external shuffle service, please also upgrade external shuffle service to use adaptive execution feature.
```

### Environment variables

### log4j.properties

Use options below to overwrite log4j.properties of driver/executor:
```
--conf 'spark.driver.extraJavaOptions=-Dlog4j.configuration=file:"./lib/log4j-spark2.properties"

--files ./lib/log4j-spark2.properties
--conf'spark.executor.extraJavaOptions=-Dlog4j.configuration=log4j-spark2.properties'
```