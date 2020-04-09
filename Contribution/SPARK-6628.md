> [ClassCastException occurs when executing sql statement "insert into" on hbase table](https://issues.apache.org/jira/browse/SPARK-6628)

This happend when :
1. You created a Hive external table point to Hbase
2. Try to `INSERT INTO` this table in SparkSQL.

Hive community fixed this issue in version 4.0.0 (https://issues.apache.org/jira/browse/HIVE-20678).

CDH 5.15.2 include this patch due to http://archive.cloudera.com/cdh5/cdh/5/hive-1.1.0-cdh5.15.2.releasenotes.html which is not included in Apache Hive 1.1.0.

There is no fix in Spark side because this PR(https://github.com/apache/spark/pull/18127) is not merged into any version branch. So it really depends on if your Hive dependency included the patch of HIVE-20678, if not, this bug would still exist.