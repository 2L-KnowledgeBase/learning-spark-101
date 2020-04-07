#### [SPARK-6628 ClassCastException occurs when executing sql statement "insert into" on hbase table](https://issues.apache.org/jira/browse/SPARK-6628)

> refs:
[**CSDN** HiveHBaseTableOutputFormat cannot be cast to org.apache.hadoop.hive.ql.io.HiveOutputFormat解决方法](https://blog.csdn.net/fuhtead/article/details/81635326)

#### [SPARK-23128 A new approach to do adaptive execution in Spark SQL](https://issues.apache.org/jira/browse/SPARK-23128)

> https://github.com/Intel-bigdata/spark-adaptive

#### [SPARK-20937 Describe spark.sql.parquet.writeLegacyFormat property in Spark SQL, DataFrames and Datasets Guide](https://issues.apache.org/jira/browse/SPARK-20937)

1. [parquet.io.ParquetDecodingException: Can not read value at 0 in block -1 in file](https://stackoverflow.com/questions/37829334/parquet-io-parquetdecodingexception-can-not-read-value-at-0-in-block-1-in-file)
        ```
        add --conf "spark.sql.parquet.writeLegacyFormat=true" with spark-submit
        ```
2. fixed in 2.4.0


#### Not Sure if it's a Bug

1. unsupportted `IN` usage:

``` sql
        select distinct
                b.company_id
        from src_security a
        inner join compy_basicinfo b
        on a.company_id=b.company_id and b.is_del=0
        inner join bond_basicinfo c
        on a.secinner_id=c.secinner_id and c.isdel=0 and b.company_st=1 and b.country='CN'
        --and c.security_type_id in (select constant_id from lkp_charcode where constant_type=201 and constant_cd like '060007%')
        left join lkp_charcode lkp
        on c.security_type_id = lkp.constant_id and constant_type=201 and constant_cd like '060007%'
        where lkp.constant_id is null
 
 org.apache.spark.sql.AnalysisException: Table or view not found: `lkp_charcode`; line 50 pos 56;
 ```

2. `org.codehaus.commons.compiler.CompileException: File 'generated.java', Line 855, Column 28: Redefinition of parameter "agg_expr_11"`, [SPARK-23986](https://issues.apache.org/jira/browse/SPARK-23986)
