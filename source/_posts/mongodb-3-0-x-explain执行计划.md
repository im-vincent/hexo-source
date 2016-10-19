title: mongodb 3.0.x explain执行计划
date: 2015-10-08 15:08:43
tags: [mongodb]
---

> 本文参考MongoDB干货系列 http://www.mongoing.com/eshu_explain3

### 现版本explain有三种模式，分别如下：

- queryPlanner

- executionStats

- allPlansExecution

#### IndexFilter
IndexFilter决定了查询优化器对于某一类型的查询将如何使用index，indexFilter仅影响查询优化器对于该类查询可以用尝试哪些index的执行计划分析，查询优化器还是根据分析情况选择最优计划。

> http://docs.mongodb.org/manual/reference/command/planCacheSetFilter/#dbcmd.planCacheSetFilter


#### Stage的意义

如explain.queryPlanner.winningPlan.stage和explain.queryPlanner.winningPlan.inputStage等。

文档中仅有如下几类介绍

- COLLSCAN	全表扫描

- IXSCAN	索引扫描

- FETCH		根据索引去检索指定document

- SHARD_MERGE	将各个分片返回数据进行merge



但是根据源码中的信息，个人还总结了文档中没有的如下几类(常用如下，由于是通过源码查找，可能有所遗漏)

- SORT			表明在内存中进行了排序（与老版本的scanAndOrder:true一致）

- LIMIT			使用limit限制返回数

- SKIP			使用skip进行跳过

- IDHACK	 	针对_id进行查询

- SHARDING_FILTER	通过mongos对分片数据进行查询

- COUNT			利用db.coll.explain().count()之类进行count运算

- COUNTSCAN		count不使用用Index进行count时的stage返回

- COUNT_SCAN	count使用了Index进行count时的stage返回

- SUBPLA		未使用到索引的$or查询的stage返回

- TEXT			使用全文索引进行查询时候的stage返回

- PROJECTION	限定返回字段时候stage的返回

#### 对Explain返回逐层分析
```
"executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 29861,
                "executionTimeMillis" : 66948,
                "totalKeysExamined" : 29861,
                "totalDocsExamined" : 29861,
                "executionStages" : {
                        "stage" : "FETCH",
                        "nReturned" : 29861,
                        "executionTimeMillisEstimate" : 66244,
                        "works" : 29862,
                        "advanced" : 29861,
                        "needTime" : 0,
                        "needFetch" : 0,
                        "saveState" : 2934,
                        "restoreState" : 2934,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 29861,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 29861,
                                "executionTimeMillisEstimate" : 290,
                                "works" : 29862,
                                "advanced" : 29861,
                                "needTime" : 0,
                                "needFetch" : 0,
                                "saveState" : 2934,
                                "restoreState" : 2934,
                                ...

```

1. executionTimeMillis
	- executionStats.executionTimeMillis 	
	该query的整体查询时间
	- executionStats.executionStages.executionTimeMillis 	该查询根据index去检索document获取29861条具体数据的时间
	- executionStats.executionStages.inputStage.executionTimeMillis	该查询扫描29861行index所用时间
2. index与document扫描数与查询返回条目数
	- nReturned=totalKeysExamined & totalDocsExamined=0
	- nReturned=totalKeysExamined=totalDocsExamined(需要具体情况具体分析)
	- 如果有sort的时候，为了使得sort不在内存中进行，我们可以在保证nReturned=totalDocsExamined
的基础上，totalKeysExamined可以大于totalDocsExamined与nReturned，因为量级较大的时候内存排序非常消耗性能。
3. Stage状态分析
	- 我们希望看到的组合：
		- Fetch+IDHACK
		- Fetch+ixscan
		- Limit+（Fetch+ixscan）
		- PROJECTION+ixscan
		- SHARDING_FILTER+ixscan
	- 不希望看到包含如下的stage：
		- COLLSCAN（全表扫）
		- SORT（使用sort但是无index）
		- 不合理的SKIP
		- SUBPLA(未用到index的$or)
	- 对于count查询
		- COUNT_SCAN(好的)
		- COUNTSCAN(不好的)

当查询覆盖精确匹配，范围查询与排序的时候
{精确匹配字段,排序字段,范围查询字段}这样的索引排序会更为高效。
