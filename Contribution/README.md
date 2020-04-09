### Contribute to Apache Spark Community
 
- Github repo of Apache Spark (*ASF, Apache Software Foundation*): https://github.com/apache/spark . [Contribute it with your PR](https://github.com/apache/spark/blob/master/CONTRIBUTING.md)!
- Submit a bug at https://issues.apache.org/jira/projects/SPARK/summary .


### Known issues

#### - [1.6.3] DataframeWriter jdbc() `java.lang.IllegalArgumentException: Can't get JDBC type for null`

```
https://spark.apache.org/docs/1.6.3/api/java/org/apache/spark/sql/DataFrame.html

df.printSchema()

root
 |-- colA: string (nullable = true)
 |-- colB: null (nullable = true)
 |-- status: integer (nullable = true)

Dataframe是非类型安全的，Row的“数据结构”被保存在schema中, 由于schema中colB的FieldType为null, 因此找不到对应的jdbc字段类型
```

#### - `java.util.concurrent.TimeoutException: Futures timed out after [300 seconds]`

This usually happened when Spark try to do Broadcast HashJoin. You could solve this by:
- Increase `spark.sql.broadcastTimeout` (default 300 seconds), e.g. `conf.set("spark.sql.broadcastTimeout", 36000)`
- Avoid **Broadcast HashJoin**, maybe increase `spark.sql.autoBroadcastJoinThreshold` (default 10485760 - 10M) or remove BHJ join hits `/*+ BROADCAST(r) */`

#### - `WARN util.Utils: Truncated the string representation of a plan since it was too large. This behavior can be adjusted by setting 'spark.debug.maxToStringFields' in SparkEnv.conf.`

You might get this warning if you used `ds.describe()` and it can be safely ignored.

#### - `org.apache.spark.sql.AnalysisException: due to data type mismatch: The given values of function map should all be the same type, but they are [decimal(38,0), array<map<string,string>>];`

You should use `named_struct()` instead of `map()`, **the type of key/value in a Map should be same.**

#### - `org.apache.spark.sql.AnalysisException: Undefined function: 'xxx'. This function is neither a registered temporary function nor a permanent function registered in the database 'default'.;`

If you want to leverage a Hive UDF in SparkSQL. You can try:
1. Check if function exist with HiveExternalCatalog
    - https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-HiveExternalCatalog.html
    - https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-ExternalCatalog.html#getFunction

2. If not exits, `sparkSession.sql("""CREATE TEMPORARY FUNCTION myFunc AS 'com.facebook.hive.udf.UDFNumberRows'""")`

Also, I still recommend you to use native UDF support of SparkSQL.
- https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-udfs.html
- https://docs.databricks.com/spark/latest/spark-sql/udf-scala.html


#### - [java.lang.ClassCastException: org.apache.hadoop.hive.hbase.HiveHBaseTableOutputFormat cannot be cast to org.apache.hadoop.hive.ql.io.HiveOutputFormat](SPARK-6628.md)

#### - [parquet.io.ParquetDecodingException: Can not read value at 0 in block -1 in file](./SPARK-20937.md)

#### - `org.codehaus.commons.compiler.CompileException: File 'generated.java', Line 855, Column 28: Redefinition of parameter "agg_expr_11"` 