### [Spark SQL / Catalyst 内部原理 与 RBO](http://www.jasongj.com/spark/rbo/)

![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql.png)

从上图可见，无论是直接使用 SQL 语句还是使用 DataFrame，都会经过如下步骤转换成 DAG 对 RDD 的操作:

1. Parser 解析 SQL( 使用 Antlr 进行记法和语法解析)，生成 Unresolved Logical Plan

![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql_parser.png)

Spark SQL 解析出的 UnresolvedPlan 如下所示
```
== Parsed Logical Plan ==
'Project [unresolvedalias('sum('v), None)]
+- 'SubqueryAlias tmp
   +- 'Project ['score.id, (((100 + 80) + 'score.math_score) + 'score.english_score) AS v#493]
      +- 'Filter (('people.id = 'score.id) && ('people.age > 10))
         +- 'Join Inner
            :- 'UnresolvedRelation `people`
            +- 'UnresolvedRelation `score`
```

2. 由 Analyzer 结合 Catalog 信息生成 Resolved Logical Plan

![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql_analyzer.png)

3. Optimizer根据预先定义好的规则(RBO, Rule Based Optimization)对 Resolved Logical Plan 进行优化并生成 Optimized Logical Plan

> 举例说明一些常见的Rule:

- PushdownPredicate 是最常见的用于减少参与计算的数据量的方法
  ![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql_predicate_pushdown.png)
- ConstantFolding 常量合并，从而减少不必要的计算
  ![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql_constant_folding.png)
- ColumnPruning 将 Project 下推，在扫描表时就只筛选出满足后续操作的最小字段集(在物理上，Project 下推后，对于列式存储，如 Parquet 和 ORC，可在扫描表时就只扫描需要的列而跳过不需要的列，进一步减少了扫描开销，提高了执行速度)
  ![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql_column_pruning.png)
  
4. Query Planner 将 Optimized Logical Plan 转换成多个 Physical Plan

![](http://www.jasongj.com/img/spark/spark2_rbo/spark_sql_physical_plan.png)

5. CBO(Cost Based Optimization) 根据 Cost Model 算出每个 Physical Plan 的代价并选取代价最小的 Physical Plan 作为最终的 Physical Plan
> http://www.jasongj.com/spark/cbo/

6. Spark 以 DAG 的方法执行上述 Physical Plan
7. 在执行 DAG 的过程中，[Adaptive Execution](https://github.com/genghuiluo/Eat-Your-Own-Dog-Food/blob/master/ARTS/article/SparkSQL-AdaptiveExecution.md) 根据运行时信息动态调整执行计划从而提高执行效率


### [Spark SQL 性能优化再进一步 CBO 基于代价的优化](http://www.jasongj.com/spark/cbo/)
