## Spark SQL syntax

**SparkSQL follows Hive style**, so you can refer to Hive Syntax for better [documentation](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select).
The supported and unsupported Hive features by SparkSQL can be found in the official documentation **according to different Spark version**(e.g. [lastest version](http://spark.apache.org/docs/latest/sql-programming-guide.html#supported-hive-features)).

> https://docs.databricks.com/spark/latest/spark-sql/index.html

**Missing Keyword in the doc of databricks above**

1. `INTERSECT` / `EXCEPT` / `MINUS` 
	
- `INTERSECT`：returns any distinct values that are returned by both the query on the left and right sides of the INTERSECT operator, 左表和右表去重后的交集
- `EXCEPT` / `MINUS`: returns any distinct values from the query left of the EXCEPT operator that aren't also found on the right query, 存在于左表但是不存在于右表去重后的差集
- [EXCEPT(T-SQL keyword)](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/set-operators-except-and-intersect-transact-sql?view=sql-server-2017) and [MINUS(PL/SQL keyword)](https://docs.oracle.com/cd/B19306_01/server.102/b14200/queries004.htm) are same in Spark SQL: [SPARK-10601 Spark SQL - Support for MINUS](https://issues.apache.org/jira/browse/SPARK-10601)


2. [`CLUSTER BY` / `DISTRIBUTE BY` & `SORT BY`](https://docs.databricks.com/spark/latest/spark-sql/language-manual/select.html)

3. [Create Table](https://docs.databricks.com/spark/latest/spark-sql/language-manual/create-table.html)
	- [with Hive format](https://docs.databricks.com/spark/latest/spark-sql/language-manual/create-table.html#create-table-with-hive-format)

``` sql
-- create a external table in sparkSQL style

CREATE TABLE test_crs (
colA bigint,
colB string )
USING PARQUET 
PARTITIONED BY (colB)
CLUSTERED BY (colA) INTO 8 buckets
LOCATION "/apps/dt/gops/tmp/rsk_crs/test"
-- LOCATION: The created table uses the specified directory to store its data. This clause automatically implies EXTERNAL.

/*
org.apache.hadoop.security.AccessControlException: Permission denied: user=pp_dt_gops_batch_dev, access=WRITE, inode="/user/hive/warehouse/test_crs-__PLACEHOLDER__":hive:supergroup:drwxr-xr-x

The reason of this failure is straight forward. According to this part of Spark source code https://github.com/apache/spark/blob/master/sql/hive/src/main/scala/org/apache/spark/sql/hive/HiveExternalCatalog.scala
val tempPath = {
val dbLocation = new Path(getDatabase(tableDefinition.database).locationUri)
new Path(dbLocation, tableDefinition.identifier.table + "-__PLACEHOLDER__")
}
 
Since restriction of Hive metastore, spark has to create a temporary HDFS directory (“target_schema_hdfs_path” + “table_name” + “-__PLACEHOLDER__”).
In our case, schema is “default” and it’s hdfs path is “/user/hive/warehouse”, while gops_batch_dev doesn’t have a write permission for this. So we failed.
 
###How solve this problem?###
 
Start spark-sql with a specified warehouse dir install of default “/user/hive/warehouse” (you can set any persistent hdfs path which gops_batch_dev is able to write)
$ spark-sql --conf spark.ui.port=12345 --conf spark.sql.warehouse.dir=/apps/dt/gops/tmp/rsk_crs/test
 
Create a dummy schema (named anything you what, we can consider a standard here..)
spark-sql> create database dummy;
 
Create table under the schema you just created;
spark-sql> CREATE TABLE dummy.test_crs (
colA bigint,
colB string )
USING PARQUET
PARTITIONED BY (colB)
CLUSTERED BY (colA) INTO 8 buckets
LOCATION "/apps/dt/gops/tmp/rsk_crs/test";
 
Exit spark-sql and reopen a spark-sql to check if we are good
*/
```

**Difference in Different Version, use 1.6.3 & 2.30 as a example**

> spark2.3.0

1. suport `EXISTS` & `IN`
2. support `UNION`
3. `collect_set()` cannot have map type data, e.g. `collect_set(map("reportYear",report_year)`, use `collect_list` instead

> spark1.6.3

1. [SparkSQL doesn't currently have EXISTS & IN](https://stackoverflow.com/questions/34861516/spark-replacement-for-exists-and-in)
	```
	org.apache.spark.sql.AnalysisException: Unsupported language features in query
	```
2. not suport `UNION`, You must use `UNION ALL` instead.
	```
	org.apache.spark.sql.AnalysisException: missing ALL at 'select' near '<EOF>';
	```

## Spark SQL built-in functions

> https://spark.apache.org/docs/2.3.0/api/sql/index.html

#### *references*

1. [Dynamic Partition Inserts](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-dynamic-partition-inserts.html)

