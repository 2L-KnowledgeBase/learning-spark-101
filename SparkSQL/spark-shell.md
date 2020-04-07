## Spark Shell

`spark2-shell --master local[4] --jars ./mongo-java-driver-3.9.0.jar,./mongo-spark-connector_2.11-2.3.0.jar`, other configurations introduced in [spark configuration](https://github.com/genghuiluo/Monkey-D-Luffy/blob/master/TechStack/bigdata-spark/spark-configuration.md)

```
# import package
scala> import com.mongodb.spark.MongoSpark
import com.mongodb.spark.MongoSpark

# impot multiple package
scala> import com.mongodb.spark._,org.apache.spark.api.java._

# multi-line string scala> spark.sql("""select 1 as
     | a """)

### DataFrameReader jdbc()
scala> val prop = new java.util.Properties
scala> val url = "jdbc:oracle:thin:@//10.100.xx.xx:1521/orcl" //jdbc url

scala> prop.setProperty("driver","oracle.jdbc.driver.OracleDriver") //jdbc class, include jdbc jar with --jars
scala> prop.setProperty("user","test")
scala> prop.setProperty("password","1234") 

scala> val tableNm = "compy_basicinfo" scala> val df1 = spark.read.jdbc(url, tableNm, prop)
scala> val sql = "select company_id from compy_basicinfo where is_del=1" 
scala> val df2 = spark.read.jdbc(url, "(" + sql +") t", prop)



### MongoDB Spark Connector, https://docs.mongodb.com/spark-connector/master/
scala> val jsc = new JavaSparkContext(sc)
jsc: org.apache.spark.api.java.JavaSparkContext = org.apache.spark.api.java.JavaSparkContext@774304ca

scala> val readConfigMap = new java.util.HashMap[String,String]
readConfigMap: java.util.HashMap[String,String] = {}

scala> readConfigMap.put("uri","mongodb://ro_user:UojbgXRLoC@10.100.47.45:27017/public")
res1: String = null

scala> readConfigMap.put("collection","test")
res3: String = null

scala> val readConfig = ReadConfig.create(readConfigMap)
readConfig: com.mongodb.spark.config.ReadConfig = ReadConfig(public,test,Some(mongodb://ro_user:UojbgXRLoC@10.100.47.45:27017/public),1000,DefaultMongoPartitioner,Map(),15,ReadPreferenceConfig(primary,None),ReadConcernConfig(None),AggregationConfig(None,None),false,true,250,true,true)

scala> val ds =  MongoSpark.load(jsc, readConfig).toDF()
19/05/10 18:36:53 WARN sql.SparkSession$Builder: Using an existing SparkSession; some configuration may not take effect.
19/05/10 18:36:56 WARN sql.MongoInferSchema: Field '_id' contains conflicting types converting to StringType
19/05/10 18:36:59 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
ds: org.apache.spark.sql.DataFrame = [_class: string, _id: string ... 11 more fields]

scala> ds.printSchema
root
 |-- _id: struct (nullable = true)
 |    |-- oid: string (nullable = true)
 |-- company_id: integer (nullable = true)

scala> val ds1 = spark.sql("select named_struct('oid','5cda6890e2b0a0153fad3a7a') as _id, 12345 as company_id")

scala> val delsertWriteConfig = ReadConfig.create(readConfigMap)

scala> MongoSpark.save(ds1.write.mode("append"), delsertWriteConfig)

scala> MongoSpark.save(ds1.write.mode("overwrite"), overWriteConfig)



## Elasticsearch for Apache Hadoop, SparkSQL support: https://www.elastic.co/guide/en/elasticsearch/hadoop/6.5/spark.html#spark-sql
scala> import org.elasticsearch.spark.sql.api.java.JavaEsSparkSQL;

scala> val cfg = new java.util.HashMap[String,String]
scala> cfg.put("es.nodes","10.100.44.80")
scala> val df = spark.sql("select 'mark' as name, 'luo' as lastname")
scala> JavaEsSparkSQL.saveToEs(df, "accounts/person", cfg);
# result document in ES
{
    "_index": "accounts",
    "_type": "person",
    "_id": "GzQw2WoBtwYwDuqXXYkT",
    "_score": 1,
    "_source": {
        "name": "mark",
        "lastname": "luo"
    }
},

scala> val df = spark.sql("select 3 as _id, 'dora' as name, 'zhang' as lastname")
scala> JavaEsSparkSQL.saveToEs(df, "accounts/person", cfg);
Caused by: org.apache.spark.util.TaskCompletionListenerException: Could not write all entries for bulk operation [1/1]. Error sample (first [5] error messages):
        org.elasticsearch.hadoop.rest.EsHadoopRemoteException: mapper_parsing_exception: Field [_id] is a metadata field and cannot be added inside a document. Use the index API request parameters.
        {"index":{}}
{"_id":3,"name":"dora","lastname":"zhang"}

scala> cfg.put("es.mapping.id","id")
scala> val df = spark.sql("select 3 as id, 'dora' as name, 'zhang' as lastname")
scala> JavaEsSparkSQL.saveToEs(df, "accounts/person", cfg);
# result document in ES
{
    "_index": "accounts",
    "_type": "person",
    "_id": "3",
    "_score": 1,
    "_source": {
        "id": 3,
        "name": "dora",
        "lastname": "zhang"
    }
}

scala> val df = spark.sql("select 4 as id, array('dora','mark','shawn') as names, named_struct('a', 1, 'b', 2) as details")
scala> JavaEsSparkSQL.saveToEs(df, "accounts/person", cfg);
# result document in ES, automatic index and mapping 
{
    "_index": "accounts",
    "_type": "person",
    "_id": "4",
    "_score": 1,
    "_source": {
        "id": 4,
        "names": [
            "dora",
            "mark",
            "shawn"
        ],
        "details": {
            "a": 1,
            "b": 2
        }
    }
},

## ExternalCatalog:https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-ExternalCatalog.html#getDatabase
scala> val d = spark.sharedState.externalCatalog.getDatabase("default");
d: org.apache.spark.sql.catalyst.catalog.CatalogDatabase = CatalogDatabase(default,Default Hive database,hdfs://horton/user/hive/warehouse,Map())

## ShareState https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-SharedState.html#warehousePath-indepth
1. hive.metastore.warehouse.dir if defined and spark.sql.warehouse.dir is not
2. spark.sql.warehouse.dir if hive.metastore.warehouse.dir is undefined
hive.metastore.warehouse.dir property in hive-site.xml is deprecated since Spark 2.0.0. Use spark.sql.warehouse.dir to specify the default location of the databases in a Hive warehouse.





# Ctrl + D to quit
scala> :quit
```
