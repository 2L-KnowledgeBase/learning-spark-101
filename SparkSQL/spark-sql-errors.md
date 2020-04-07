### Spark SQL 常见错误

1. `java.lang.IllegalArgumentException: Can't get JDBC type for null`

``` java
// spark 1.6.3, https://spark.apache.org/docs/1.6.3/api/java/org/apache/spark/sql/DataFrame.html 
df.printSchema()

null 类型找不到对应的jdbc类型

root
 |-- compy_chattelreg_xygs_sid: string (nullable = true)
 |-- company_id: null (nullable = true)
 |-- status: integer (nullable = true)
 |-- notice_dt: date (nullable = true)
 |-- reg_dt: date (nullable = true)
 |-- reg_nuim: string (nullable = true)
 |-- debt_amt: string (nullable = true)
 |-- seq_num: integer (nullable = true)
 |-- reg_gov: string (nullable = true)
 |-- detail: string (nullable = true)
 |-- cancal_dt: date (nullable = true)
 |-- mortgagee: string (nullable = true)
 |-- mortgagor: string (nullable = true)
 |-- start_dt: date (nullable = true)
 |-- end_dt: date (nullable = true)
 |-- pledge_nm: string (nullable = true)
 |-- pledge_amt: string (nullable = true)
```

2. `java.util.concurrent.TimeoutException: Futures timed out after [300 seconds]`

This happens because Spark tries to do Broadcast Hash Join and **one of the DataFrames is very large**, so sending it consumes much time.

You can:
- Set higher spark.sql.broadcastTimeout to increase timeout - spark.conf.set("spark.sql.broadcastTimeout",  newValueForExample36000)
- persist() both DataFrames, then Spark will use Shuffle Join - reference from [this](https://stackoverflow.com/questions/36290486/spark-job-restarted-after-showing-all-jobs-completed-and-then-fails-timeoutexce)

3. `WARN util.Utils: Truncated the string representation of a plan since it was too large. This behavior can be adjusted by setting 'spark.debug.maxToStringFields' in SparkEnv.conf.`

4. `org.apache.spark.sql.AnalysisException: due to data type mismatch: The given values of function map should all be the same type, but they are [decimal(38,0), array<map<string,string>>];`

You should use `named_struct()` instead of `map()`; **Map's key/value datatype should be same.**

5. `org.apache.spark.sql.AnalysisException: Undefined function: 'format_person_nm'. This function is neither a registered temporary function nor a permanent function registered in the database 'default'.;`

```
https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-HiveExternalCatalog.html
https://jaceklaskowski.gitbooks.io/mastering-spark-sql/spark-sql-ExternalCatalog.html#getFunction
sparkSession.sql("""CREATE TEMPORARY FUNCTION myFunc AS 'com.facebook.hive.udf.UDFNumberRows'""")
```

6. `org.apache.spark.sql.AnalysisException: Function to_json already exists;`
7. [`java.util.concurrent.TimeoutException: Futures timed out after [300 seconds]`](https://stackoverflow.com/questions/41123846/why-does-join-fail-with-java-util-concurrent-timeoutexception-futures-timed-ou)
8. [`Can't zip RDDs with unequal numbers of partitions: List(m, n)`](https://stackoverflow.com/questions/29814499/cant-zip-rdds-with-unequal-numbers-of-partitions/29847091)

```
java.lang.IllegalArgumentException: Can't zip RDDs with unequal numbers of partitions: List(13, 500)
        at org.apache.spark.rdd.ZippedPartitionsBaseRDD.getPartitions(ZippedPartitionsRDD.scala:57)
        at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:253)
        at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:251)
        at scala.Option.getOrElse(Option.scala:121)
        at org.apache.spark.rdd.RDD.partitions(RDD.scala:251)
        at org.apache.spark.rdd.MapPartitionsRDD.getPartitions(MapPartitionsRDD.scala:35)
        at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:253)
        at org.apache.spark.rdd.RDD$$anonfun$partitions$2.apply(RDD.scala:251)
        at scala.Option.getOrElse(Option.scala:121)
        at org.apache.spark.rdd.RDD.partitions(RDD.scala:251)
        at org.apache.spark.sql.execution.datasources.FileFormatWriter$.write(FileFormatWriter.scala:193)
        at org.apache.spark.sql.execution.datasources.InsertIntoHadoopFsRelationCommand.run(InsertIntoHadoopFsRelationCommand.scala:154)
        at org.apache.spark.sql.execution.datasources.DataSource.writeAndRead(DataSource.scala:528)
        at org.apache.spark.sql.execution.command.CreateDataSourceTableAsSelectCommand.saveDataIntoTable(createDataSourceTables.scala:216)
        at org.apache.spark.sql.execution.command.CreateDataSourceTableAsSelectCommand.run(createDataSourceTables.scala:176)
        at org.apache.spark.sql.execution.command.DataWritingCommandExec.sideEffectResult$lzycompute(commands.scala:104)
        at org.apache.spark.sql.execution.command.DataWritingCommandExec.sideEffectResult(commands.scala:102)
        at org.apache.spark.sql.execution.command.DataWritingCommandExec.doExecute(commands.scala:122)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:136)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:132)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeQuery$1.apply(SparkPlan.scala:160)
        at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
        at org.apache.spark.sql.execution.SparkPlan.executeQuery(SparkPlan.scala:157)
        at org.apache.spark.sql.execution.SparkPlan.execute(SparkPlan.scala:132)
        at org.apache.spark.sql.execution.adaptive.QueryStage.executeStage(QueryStage.scala:160)
        at org.apache.spark.sql.execution.adaptive.QueryStage.doExecute(QueryStage.scala:169)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:136)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$execute$1.apply(SparkPlan.scala:132)
        at org.apache.spark.sql.execution.SparkPlan$$anonfun$executeQuery$1.apply(SparkPlan.scala:160)
        at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:151)
        at org.apache.spark.sql.execution.SparkPlan.executeQuery(SparkPlan.scala:157)
        at org.apache.spark.sql.execution.SparkPlan.execute(SparkPlan.scala:132)
        at org.apache.spark.sql.execution.QueryExecution.toRdd$lzycompute(QueryExecution.scala:81)
        at org.apache.spark.sql.execution.QueryExecution.toRdd(QueryExecution.scala:81)
        at org.apache.spark.sql.DataFrameWriter$$anonfun$runCommand$1.apply(DataFrameWriter.scala:654)
        at org.apache.spark.sql.DataFrameWriter$$anonfun$runCommand$1.apply(DataFrameWriter.scala:654)
        at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:77)
        at org.apache.spark.sql.DataFrameWriter.runCommand(DataFrameWriter.scala:654)
        at org.apache.spark.sql.DataFrameWriter.createTable(DataFrameWriter.scala:458)
        at org.apache.spark.sql.DataFrameWriter.saveAsTable(DataFrameWriter.scala:437)
        at org.apache.spark.sql.DataFrameWriter.saveAsTable(DataFrameWriter.scala:393)
        at tempo.spark2.LoadHive.overwrite(LoadHive.java:63)
        at tempo.spark2.DriverMain.main(DriverMain.java:131)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.apache.spark.deploy.JavaMainApplication.start(SparkApplication.scala:52)
        at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:879)
        at org.apache.spark.deploy.SparkSubmit$.doRunMain$1(SparkSubmit.scala:197)
        at org.apache.spark.deploy.SparkSubmit$.submit(SparkSubmit.scala:227)
        at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:136)
        at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
```
