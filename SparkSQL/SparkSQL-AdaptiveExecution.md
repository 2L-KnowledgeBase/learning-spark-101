### [Adaptive Execution 让 Spark SQL 更高效更智能](http://www.jasongj.com/spark/adaptive_execution/)

> RBO & CBO ( [ARTS/article/SparkSQL-QE内部原理.md](https://github.com/genghuiluo/Eat-Your-Own-Dog-Food/blob/master/ARTS/article/SparkSQL-QE%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86.md) ) , 从 *查询本身* 与 *目标数据的特点* 的角度尽可能保证了最终生成的执行计划的高效性

But:

1. 执行计划一旦生成，便不可更改，即使执行过程中发现后续执行计划可以进一步优化，也只能按原计划执行
2. CBO 基于统计信息生成最优执行计划，需要提前生成统计信息，成本较大? 且不适合数据更新频繁的场景?
3. CBO 基于基础表的统计信息与操作对数据的影响推测中间结果的信息，只是估算(org.apache.spark.util.SizeEstimator,  returns the number of bytes an object takes up on the JVM heap)，不够精确(大部分时候都不太准确) 

Adaptive Execution 将可以根据执行过程中的中间数据优化后续执行，从而提高整体执行效率。核心在于两点:

1. 执行计划可动态调整
2. 调整的依据是中间结果的精确统计信息 ( Runtime ) 


#### Feature 1: Dynamicly set number of Shuffle Partition

原有 Shuffle 的问题: 使用 Spark SQL 时，可通过 spark.sql.shuffle.partitions 指定 Shuffle 时 Partition 个数(默认200)，也即 Reducer (代指 Spark Shuffle 过程中执行 Shuffle Read 的 Task) 个数

- 单次 Shuffle 的 Partition 个数不易过大(过多的Task调度开销/小文件过多), 不易过小(并发低导致执行效率低/单个Task由于数据量过大导致GC、Spill到磁盘)
- 不同 Stage 之间的 Shuffle 对应的数据量不一样，因此最优的 Partition 个数也不一样。使用统一的 Partition 个数很难保证所有 Shuffle 都最优

![](http://www.jasongj.com/img/spark/spark4_ae/spark_ae_fix_reducer_detail.png)

With AE enabled (可通过 spark.sql.adaptive.enabled=true 启用 Adaptive Execution 从而启用自动设置 Shuffle Reducer 这一特性):

1. Shuffle Write 结束后，根据各 Mapper 输出，统计得到各 Partition 的数据量
2. 通过 ExchangeCoordinator 计算出合适的 post-shuffle Partition 个数（即 Reducer）个数, targetPostShuffleInputSize 默认为 64MB，每个 Reducer 读取数据量不超过 64MB
3. 启动相应个数的 Reducer 任务
4. 每个 Reducer 读取一个或多个 Shuffle Write Partition 数据, 只结合相临的 Partition，从而保证顺序读，提高磁盘 IO 性能

![](http://www.jasongj.com/img/spark/spark4_ae/spark_ae_auto_reducer_detail_1.png)


#### Feature 2: Dynamicly tune Execution Plan

在不开启 Adaptive Execution 之前，执行计划一旦确定，即使发现后续执行计划可以优化，也不可更改

如下图所示，SortMergJoin 的 Shuffle Write 结束后，发现 Join 一方的 Shuffle 输出只有 46.9KB，仍然继续执行 SortMergeJoin

![](http://www.jasongj.com/img/spark/spark4_ae/spark_ae_fix_dag.png)

With AE enabled (当 spark.sql.adaptive.enabled 与 spark.sql.adaptive.join.enabled 都设置为 true 时，开启 Adaptive Execution 的动态调整 Join 功能):

- Shuffle Write 结束后，可从每个 ShuffleMapTask 的 MapStatus 中统计得到按原计划执行时 Stage 2 各 Partition 的数据量以及 Stage 2 需要读取的总数据量

[关于sparkSQL 3种join的实现方式]()

#### Feature 3: Automatically handle Skew Join



> refs:

1. [Video: Carson Wang(Intel) "An Adaptive Execution Engine For Apache Spark SQL - Carson Wang" @SparkSubmit](https://www.youtube.com/watch?v=FZgojLWdjaw)
2. [Intel-bigdata/spark-adaptive @github](https://github.com/Intel-bigdata/spark-adaptive)
3. [Spark SQL在100TB上的自适应执行实践](https://mp.weixin.qq.com/s?__biz=MzA4Mzc0NjkwNA==&mid=2650784030&idx=1&sn=2c61e166b535199ee53e579a5092ff80&chksm=87faa829b08d213f55dab289bf5a12cfe376be0c944e03279a1c93e0f0d2164f1c6a6c7c880a&mpshare=1&scene=1&srcid=0111fEEzMCuhKozD4hsN4EE5&pass_ticket=WwOAQGxxBX9z63UyuFIXnWVm%2FSJhHkYwdsKplVDbaiA66ueqnDOtzgq86NgTgqvt#rd)

#### 自适应执行架构

1. 在Spark SQL中，当Spark确定最后的物理执行计划后，根据每一个operator对RDD的转换定义，它会生成一个RDD的DAG图。之后Spark基于DAG图静态划分stage并且提交执行，所以一旦执行计划确定后，在运行阶段无法再更新。
2. 自适应执行的基本思路是在执行计划中事先划分好stage，然后按stage提交执行，在运行时收集当前stage的shuffle统计信息，以此来优化下一个stage的执行计划，然后再提交执行后续的stage。

![](https://mmbiz.qpic.cn/mmbiz_png/wvkocF2MXjWndLdS72JOqymLBJU99EfEEeXYwMq7ZRRibKg57wOYXLtvOdhjxc2jI1C4Xgwypv3CoUrz2E6KkeQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从上图中我们可以看出自适应执行的工作方法:
1. 以Exchange节点作为分界将执行计划这棵树划分成多个QueryStage（Exchange节点在Spark SQL中代表shuffle）
2. 每一个QueryStage都是一棵独立的子树，也是一个独立的执行单元。在加入QueryStage的同时，我们也加入一个QueryStageInput的叶子节点，作为父亲QueryStage的输入。例如对于图中两表join的执行计划来说我们会创建3个QueryStage。最后一个QueryStage中的执行计划是join本身，它有2个QueryStageInput代表它的输入，分别指向2个孩子的QueryStage。
3. 在执行QueryStage时，我们首先提交它的孩子stage，并且收集这些stage运行时的信息。当这些孩子stage运行完毕后，我们可以知道它们的大小等信息，以此来判断QueryStage中的计划是否可以优化更新。(例如当我们获知某一张表的大小是5M，它小于broadcast的阈值时，我们可以将SortMergeJoin转化成BroadcastHashJoin来优化当前的执行计划。我们也可以根据孩子stage产生的shuffle数据量，来动态地调整该stage的reducer个数。)
4. 在完成一系列的优化处理后，最终我们为该QueryStage生成RDD的DAG图，并且提交给DAG Scheduler来执行。

https://issues.apache.org/jira/browse/SPARK-23128