@[TOC](MongoDB 执行计划字段解释)

## 1. 阶段操作描述

- COLLSCAN 集合扫描
- IXSCAN 索引扫描
- FETCH 检出文档
- SHARD_MERGE 合并分片中结果
- SHARDING_FILTER 分片中过滤掉孤立文档
- LIMIT 使用limit 限制返回数
- PROJECTION 使用 skip 进行跳过
- IDHACK 针对_id进行查询
- COUNT 利用db.coll.explain().count()之类进行count运算
- COUNTSCAN count不使用Index进行count时的stage返回
- COUNT_SCAN count使用了Index进行count时的stage返回
- SUBPLA 未使用到索引的$or查询的stage返回
- TEXT 使用全文索引进行查询时候的stage返回
- PROJECTION 限定返回字段时候stage的返回



## 2. queryPlanner

- queryPlanner.namespace 一个字符串，指定运行查询的命名空间（即.）。
- queryPlanner.indexFilterSet boolan值，表示MongoDB 对于此query shape 是否使用了索引过滤器。
- queryPlanner.winningPlan 文档类型，详细显示查询优化程序选择的查询计划。
- winningPlan.stage 阶段名称。每个阶段都有每个阶段特有的信息。 例如，IXSCAN 阶段将包括索引边界以及特定于索引扫描的其他数据。 如果阶段具有子阶段或多个子阶段，则阶段将具有inputStage 或 inputStages。
- winningPlan.inputStage 描述子阶段的文档。它为其父级提供文档或索引键。 如果父级只有一个子级，则该字段存在。
- winningPlan.inputStages 描述子阶段的数组。子阶段为父阶段提供文档或索引键。 如果父级具有多个子节点，则该字段存在。 例如，$or 表达式或索引交集的阶段消耗来自多个源的输入。
- queryPlanner.rejectedPlans 查询优化器考虑和拒绝的候选计划数组。 如果没有其他候选计划，则该数组可以为空。



## 3. executionStats 

- executionStats 描述获胜计划完整的查询执行信息。 对于写入操作，是指将要执行修改的信息，但不会真实修改数据库。

- executionStats.nReturned 查询条件匹配到的文档数量。

- **executionStats.executionTimeMillis 查询计划选择和查询执行所需的总时间（以毫秒为单位）。executionTimeMillis 对应于早期版本的MongoDB中cursor.explain() 返回的millis字段**。

- **executionStats.totalKeysExamined 扫描的索引条目数。totalKeysExamined 对应于早期版本的MongoDB中cursor.explain() 返回的 nscanned字段**。

- **executionStats.totalDocsExamined 扫描的文档数。totalDocsExamined 对应于早期版本的MongoDB中cursor.explain() 返回的 nscannedObjects字段**。

- executionStats.executionStages 阶段树形式展示获胜计划完整的执行信息。即，一个阶段可以有一个inputStage或多个inputStages。

- executionStats.executionStages.works 查询执行阶段执行的“工作单元”的数量。 查询执行阶段将其工作分为小单元。 “工作单位”可能包括检查单个索引键，从集合中提取单个文档，将投影应用于单个文档或执行内部记账。

  

- executionStats.executionStages.advanced 返回到父阶段的结果数。

- executionStats.executionStages.needTime 未将中间结果返回给其父级的工作循环数。 例如，索引扫描阶段可以花费一个工作周期来寻找索引中的新位置而不是返回索引关键字; 这个工作周期将计入explain.executionStats.executionStages.needTime而不是explain.executionStats.executionStages.advanced。

  

- executionStats.executionStages.needYield 存储层请求查询系统产生锁定的次数。

  

- executionStats.executionStages.isEOF 执行阶段是否已到达流的结尾：

  - 如果为true或1，则执行阶段已到达流末尾。
  - 如果为false或0，则阶段可能仍会返回结果。 例如，有限制的查询，其执行阶段包含LIMIT阶段，其中查询的输入阶段为IXSCAN。 如果查询返回超过指定的限制，LIMIT阶段将报告isEOF：1，但其基础IXSCAN阶段将报告isEOF：0。

- executionStats.executionStages.inputStage.keysExamined 对于扫描索引的查询执行阶段（例如IXSCAN），keysExamined是在索引扫描过程中检查的入站和越界键的总数。 如果索引扫描由单个连续范围的键组成，则只需要检查入站键。 如果索引边界由若干键范围组成，则索引扫描执行过程可以检查越界键，以便从一个范围的末尾跳到下一个范围的开头。

