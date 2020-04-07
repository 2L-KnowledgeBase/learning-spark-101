### Spark Job 性能调优

> refs: https://www.zybuluo.com/xiaop1987/note/76737 & https://www.zybuluo.com/xiaop1987/note/102894

#### 1. Spark 是如何执行程序的 

keywords: transformation，action，RDD, job, stage, task

**一个 Spark 应用包括一个 driver 进程和若干个分布在集群的各个节点上的 executor 进程。**

什么决定数据是否需要 shuffle ？*narrow transformation vs wide transformation*


#### 2. 选择正确的 Operator
 
当需要使用 Spark 完成某项功能时，程序员需要从不同的 action 和 transformation 中选择不同的方案以获得相同的结果。但是不同的方案，最后执行的效率可能有云泥之别。

#### 3. 避免 Shuffle

#### 4. 调整 资源分配 & 并发

#### 5. 文件格式





### Spark SQL 性能调优

#### 1. Join 的优化
> https://blog.csdn.net/weixin_37136725/article/details/78989086

Spark将参与Join的两张表抽象为流式遍历表(streamIter)和查找表(buildIter) *通常streamIter为大表, buildIter为小表, spark根据join方式决定*

在实际计算时，spark会基于streamIter来遍历，每次取出streamIter中的一条记录rowA，根据Join条件计算keyA，然后根据该keyA去buildIter中查找所有满足Join条件(keyB==keyA)的记录rowBs，并将rowBs中每条记录分别与rowAjoin得到join后的记录，最后根据过滤条件得到最终join的记录。
由于对于每条来自streamIter的记录，都要去buildIter中查找匹配的记录，所以buildIter一定要是查找性能较优的数据结构(hash table)

1. inner join: 根据估算大小，大表为streamIter, 小表为buildIter
2. left join: 左表为streamIter, 右表为buildIter
3. right join: 右表为streamIter, 左表为buildIter
4. full join: 仅采用sort merge join实现，左边和右表既要作为streamIter，又要作为buildIter

3种join的实现:
- Sort Merge Join
	在shuffle read阶段，分别对streamIter和buildIter进行merge sort，在遍历streamIter时，对于每条记录，都采用顺序查找的方式从buildIter查找对应的记录，由于两个表都是排序的，每次处理完streamIter的一条记录后，对于streamIter的下一条记录，只需从buildIter中上一次查找结束的位置开始查找 (每次在buildIter中查找不必重头开始，最适合大表之间join)
- Broadcast Hash Join (similar to MapJoin in Hive)
	如果buildIter是一个非常小的表，那么其实就没有必要大动干戈做shuffle了，直接将buildIter广播到每个计算节点，然后将buildIter放到hash表中(当buildIter的**估计大小**不超过参数 `spark.sql.autoBroadcastJoinThreshold` 设定的值*默认10M*，那么就会自动采用broadcast join，否则采用sort merge join)
- Shuffle Hash Join (默认是关闭的)

Shuffle Hash Join 需要满足4个条件才会被spark使用:
1. `spark.sql.join.preferSortMergeJoin=false` 开启
2. buildIter总体估计大小超过spark.sql.autoBroadcastJoinThreshold设定的值，即不满足broadcast join条件
3. 每个分区的平均大小不超过spark.sql.autoBroadcastJoinThreshold设定的值，即shuffle read阶段每个分区来自buildIter的记录要能放到内存中
4. streamIter的大小是buildIter三倍以上 


#### 2. 数据倾斜 skew 

数据倾斜指的是由于数据分区不均匀导致的，spark一部分tasks承担的数据量太大，而导致整体运行时间过长的现象。
一般出现在对大表的join/group by等过程中，大表的join/group by key集中分布在某几个取值上, job在某个或某些task的处理上停留时间过长（more than 0.5 hour）

#### 3. Dataframe Reader jdbc 读取优化
> https://www.jianshu.com/p/83d273dfea1c

1. 默认地, `SparkSession.read.jdbc()` 读取并发度为1(只有1个partition)
2. 调整分区数量, 提高并发度 
	- 根据Long类型字段分区, average 
	- 根据任意类型字段分区, range

> refs:

1. [Spark SQL / Catalyst 内部原理 与 RBO](https://github.com/genghuiluo/Eat-Your-Own-Dog-Food/blob/master/ARTS/article/SparkSQL-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86.md)
2. [Adaptive Execution 让 Spark SQL 更高效更智能](https://github.com/genghuiluo/Eat-Your-Own-Dog-Food/blob/master/ARTS/article/SparkSQL-AdaptiveExecution.md)
