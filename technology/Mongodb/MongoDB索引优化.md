@[TOC](MongoDB 索引优化)

# 1. 一图看懂索引原理

![img](file:///C:/Users/wpzhou/AppData/Local/YNote/data/327058467@qq.com/ef5fa05d07374631971c5ed2fe1c24fc/clipboard.png)

```shell
# 初始化数据
> db.comment.insertMany(
... [
... {'timestamp': 3, 'anonymous': true, 'rating': 1},
... {'timestamp': 2, 'anonymous': false, 'rating': 5},
... {'timestamp': 1, 'anonymous': false, 'rating': 3},
... {'timestamp': 4, 'anonymous': false, 'rating': 2}
... ])
> db.comment.createIndex({'anonymous':1, 'rating': 1})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
> db.comment.getIndexes()
[
	{
		"v" : 2,
		"key" : {
			"_id" : 1
		},
		"name" : "_id_",
		"ns" : "test.comment"
	},
	{
		"v" : 2,
		"key" : {
			"anonymous" : 1,
			"rating" : 1
		},
		"name" : "anonymous_1_rating_1",
		"ns" : "test.comment"
	}
]
```
# 2. 查看执行计划

- 使用explain 查看执行计划，传入参数可以获取具体的执行过程，比如DocsExamined、KeysExamined

  - executionStats ：捕获获胜计划执行的相关信息。

  - allPlansExecution ：捕获选择执行计划期间执行的相关信息
- 字段解释
  - 见[MongoDB 执行计划字段解释]

```shell
> db.comment.find(  
... { timestamp: { $gte: 2, $lte: 4 }, anonymous: false }  
... ).sort( { rating: -1 }  
... ).explain()
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.comment",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"anonymous" : {
						"$eq" : false
					}
				},
				{
					"timestamp" : {
						"$lte" : 4
					}
				},
				{
					"timestamp" : {
						"$gte" : 2
					}
				}
			]
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"filter" : {
				"$and" : [
					{
						"timestamp" : {
							"$lte" : 4
						}
					},
					{
						"timestamp" : {
							"$gte" : 2
						}
					}
				]
			},
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"anonymous" : 1,
					"rating" : 1
				},
				"indexName" : "anonymous_1_rating_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"anonymous" : [ ],
					"rating" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "backward",
				"indexBounds" : {
					"anonymous" : [
						"[false, false]"
					],
					"rating" : [
						"[MaxKey, MinKey]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"serverInfo" : {
		"host" : "150501d113",
		"port" : 27017,
		"version" : "3.6.1",
		"gitVersion" : "025d4f4fe61efd1fb6f0005be20cb45a004093d1"
	},
	"ok" : 1
}

# 这个只能看到大概的执行计划，比如使用的哪个索引； 如果想要看到具体的执行过程，可使用explain的参数
> db.comment.find( { timestamp: { $gte: 2, $lte: 4 }, anonymous: false }   ).sort( { rating: -1 }   ).explain("executionStats")  
{
	"queryPlanner" : {
		"plannerVersion" : 1,
		"namespace" : "test.comment",
		"indexFilterSet" : false,
		"parsedQuery" : {
			"$and" : [
				{
					"anonymous" : {
						"$eq" : false
					}
				},
				{
					"timestamp" : {
						"$lte" : 4
					}
				},
				{
					"timestamp" : {
						"$gte" : 2
					}
				}
			]
		},
		"winningPlan" : {
			"stage" : "FETCH",
			"filter" : {
				"$and" : [
					{
						"timestamp" : {
							"$lte" : 4
						}
					},
					{
						"timestamp" : {
							"$gte" : 2
						}
					}
				]
			},
			"inputStage" : {
				"stage" : "IXSCAN",
				"keyPattern" : {
					"anonymous" : 1,
					"rating" : 1
				},
				"indexName" : "anonymous_1_rating_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"anonymous" : [ ],
					"rating" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "backward",
				"indexBounds" : {
					"anonymous" : [
						"[false, false]"
					],
					"rating" : [
						"[MaxKey, MinKey]"
					]
				}
			}
		},
		"rejectedPlans" : [ ]
	},
	"executionStats" : {
		"executionSuccess" : true,
		"nReturned" : 2,
		"executionTimeMillis" : 1,
		"totalKeysExamined" : 3,
		"totalDocsExamined" : 3,
		"executionStages" : {
			"stage" : "FETCH",
			"filter" : {
				"$and" : [
					{
						"timestamp" : {
							"$lte" : 4
						}
					},
					{
						"timestamp" : {
							"$gte" : 2
						}
					}
				]
			},
			"nReturned" : 2,
			"executionTimeMillisEstimate" : 0,
			"works" : 4,
			"advanced" : 2,
			"needTime" : 1,
			"needYield" : 0,
			"saveState" : 0,
			"restoreState" : 0,
			"isEOF" : 1,
			"invalidates" : 0,
			"docsExamined" : 3,
			"alreadyHasObj" : 0,
			"inputStage" : {
				"stage" : "IXSCAN",
				"nReturned" : 3,
				"executionTimeMillisEstimate" : 0,
				"works" : 4,
				"advanced" : 3,
				"needTime" : 0,
				"needYield" : 0,
				"saveState" : 0,
				"restoreState" : 0,
				"isEOF" : 1,
				"invalidates" : 0,
				"keyPattern" : {
					"anonymous" : 1,
					"rating" : 1
				},
				"indexName" : "anonymous_1_rating_1",
				"isMultiKey" : false,
				"multiKeyPaths" : {
					"anonymous" : [ ],
					"rating" : [ ]
				},
				"isUnique" : false,
				"isSparse" : false,
				"isPartial" : false,
				"indexVersion" : 2,
				"direction" : "backward",
				"indexBounds" : {
					"anonymous" : [
						"[false, false]"
					],
					"rating" : [
						"[MaxKey, MinKey]"
					]
				},
				"keysExamined" : 3,
				"seeks" : 1,
				"dupsTested" : 0,
				"dupsDropped" : 0,
				"seenInvalidated" : 0
			}
		}
	},
	"serverInfo" : {
		"host" : "150501d113",
		"port" : 27017,
		"version" : "3.6.1",
		"gitVersion" : "025d4f4fe61efd1fb6f0005be20cb45a004093d1"
	},
	"ok" : 1
}

```

# 3. 如何建索引
- 语法：
```shell
> db.comment.createIndex({'anonymous':1, 'rating': 1})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
```
- 针对使用比较多的查询建索引

  - 等值测试：在索引中加入所有需要做等值测试的字段，任意顺序。
  - 排序字段（多排序字段的升/降序问题 ）： 根据查询的顺序有序的向索引中添加字段。
  - 范围过滤：以字段的基数（Collection中字段的不同值的数量）从低到高的向索引中添加范围过滤字段。

# 3. 索引的优化

最佳索引的原则

1. 包含查询中所有可以做过滤及需要排序的字段
2. 任何用于范围扫描的字段以及用于排序的字段都必须在做等值查询的字段之后



通过分析执行计划，调整索引，从而让查询命中索引；

最小化docsExamined、keysExamined



# 4. 索引的选择机制

1. 
   选择出最优索引
2. 假设不存在最优索引，会做尝试，然后选择表现最好的索引；存在多条索引的情况下，MongoDB首选nscanned值最低的索引。
3. 如果存在多个不同的最佳索引，mongo 将随机选择
4. 最后优化器会记住类似查询的选择（直到大规模文件变动或者索引变动）



# 5. 优化实践

## country_themes  优化

```python
# 常用的慢查询
{"theme.themeId": theme_id}
{"theme.themeId": {"$in": theme_id_list}}


cond = {
"country": "GLOBAL", 
"theme.status": 2,
"theme.category": {"$regex": category}
}
rs = list(anthcraft.country_themes.find(cond, {"theme": 1}).sort("theme.packageTime", -1)
    
cond = {
    "country": 'GLOBAL',
    "theme.onGooglePlay": True,
    "theme.jumpToGooglePlay": True,
    "theme.packageName": {"$exists": True},
    "theme.status": 2,
    "theme.category": {"$regex": theme_category}
}
rs = list(anthcraft.country_themes.find(cond, {"theme": 1}).sort("theme.packageTime", -1)

# 优化
  db.country_themes.createIndex({ 'theme.themeId': 1}, { background: true } ) 
  # 查询速度有很多提升，慢查询log已经看不到theme.themeId相关的查询了
  
  db.country_themes.createIndex({ 'country': 1, "theme.status":1, "theme.category":1}, { background: true }) 
  
  db.country_themes.createIndex({ 'theme.packageTime': -1}, { background: true } ) 
  # 在建theme.packageTime索引之前，走country相关的索引；但是建了该索引之后，直接用theme.packageTime索引了，并没有用country索引； 所以将theme.packageTime 删了
  # 但是查询速度并没有太大的提升，跟加索引之前差不多；但是执行计划有变
  #COLLSCAN keysExamined:0 docsExamined:41535
  #IXSCAN { country: 1.0, theme.status: 1.0, theme.category: 1.0 } keysExamined:18914 docsExamined:16603
  
  db.country_themes.find({
      "country": 'GLOBAL',
      "theme.onGooglePlay": true,
      "theme.jumpToGooglePlay": true,
      "theme.packageName": {"$exists": true},
      "theme.status": 2,
      "theme.category": {"$regex": "5384154ae4b049f321413f11"}
  }).sort({"theme.packageTime": -1}).explain()
  
  > db.country_themes.count({
  ...     "country": 'GLOBAL',
  ...     "theme.status": 2
  ... })
  18913
  > db.country_themes.count()
  41670
  > db.country_themes.count({"country": 'GLOBAL'})
  33744
  
  # 怀疑$regex 并没有走索引，资料显示只有前缀型的正则表达式命中索引
  db.country_themes.find({
      "country": 'GLOBAL',
      "theme.onGooglePlay": true,
      "theme.jumpToGooglePlay": true,
      "theme.packageName": {"$exists": true},
      "theme.status": 2,
      "theme.category": {"$regex": "5384154ae4b049f321413f11"}
  })
```

  

## wallpapers  mongo 优化

```shell
# 常用的慢查询
{ $query: { status: { $in: [ 2, 3 ] }, category: /.*5c07392be4b061d99da5ddad.*/i, wallpaperId: { $lt: 21550 } }, $orderby: { weight: 1, wallpaperId: -1 } }
{ $query: { status: { $in: [ 2, 3 ] } , category: /.*530ed1c0e4b0ce13ef9146e3.*/i }, $orderby: { weight: 1, wallpaperId: -1 } }
{ $query: { status: { $in: [ 2, 3 ] }, category: /^(?!.*(5a6989a8e4b0cf38e952d122|5b5fdb76e4b0cac798f75633)).*$/, wallpaperId: { $lt: 19055 } }, $orderby: { weight: 1, wallpaperId: -1 } }

# 根据常用查询，创建索引
db.wallpapers.createIndex({ 'status': 1, 'wallpaperId': 1,'category': 1}) 

# status、wallpaperId使用索引，category未使用；只有前缀型的正则表达式命中索引
db.wallpapers.find({'status': { $in: [ 2, 3 ]}, 'wallpaperId': { $lt: 21550 }, 'category': {"$regex": ".*530ed1c0e4b0ce13ef9146e3.*"}}).sort({'weight': 1, 'wallpaperId': -1 }).explain()

# status、wallpaperId使用索引，category未使用；当前的正则不符合前缀型的正则表达式
db.wallpapers.find({'status': { $in: [ 2, 3 ]}, 'wallpaperId': { $lt: 21550 }, 'category': {"$regex": "^(?!.*(5a6989a8e4b0cf38e952d122|5b5fdb76e4b0cac798f75633)).*$"}}).sort({'weight': 1, 'wallpaperId': -1 }).explain()

# status、wallpaperId、category使用索引
db.wallpapers.find({'status': { $in: [ 2, 3 ]}, 'wallpaperId': { $lt: 21550 }, 'category': {"$regex": "^5a6989a8e4b0cf38e952d122"}}).sort({'weight': 1, 'wallpaperId': -1 }).explain()
# 未使用索引
db.wallpapers.find({'category':  "5a6989a8e4b0cf38e952d122"}).sort({'weight': 1, 'wallpaperId': -1 }).explain()

# status、category使用索引； 跳过wallpaperId任然可以用索引，这点很棒，之前以为不会用呢
db.wallpapers.find({'status': { $in: [ 2, 3 ]}, 'category': {"$regex": "^5a6989a8e4b0cf38e952d122"}}).sort({'weight': 1, 'wallpaperId': -1 }).explain()

# 但是常用查询中的category都是非前缀型的正则，所以走不了索引； 对category 不建索引
db.wallpapers.dropIndex('status_1_wallpaperId_1_category_1')
db.wallpapers.createIndex({ 'status': 1, 'wallpaperId': 1}, { background: true }) 

# 然而，速度并没有提升很多，基本没有太多的变化
# 因为原来就有索引 wallpaperId，然后status $in: [ 2, 3 ] 的记录条数20171/23986=84%
# 所以新建的索引{ 'status': 1, 'wallpaperId': 1} 并不会有明显的性能提升

# 总结： 下次见索引的时候，先分析一下目标列上的数据分布情况；太集中分布的，不推荐建索引

```



## themes mongo 优化

```shell
# 慢查询
{ $query: { themeId: { $lt: 194369 }, status: { $in: [ 2, 3 ] }, channels.10011: { $exists: false }, isShared: { $ne: 0 }, point: 0 }, $orderby: { themeId: -1 } }
{ status: { $in: [ 2, 3 ] }, isShared: { $ne: 0 }, tag: "taste", category: /^(?!.*(5384154ae4b049f321413f11)).*$/, $and: [ {}, { $or: [ { recommendTime: { $lt: new Date(1499988600000) } }, { recommendTime: new Date(1499988600000), themeId: { $lt: 3085871 } } ] } ] }, $orderby: { recommendTime: -1, themeId: -1 } }
{ themeId: { $ne: 438875 }, status: { $in: [ 2, 3 ] }, isShared: { $ne: 0 }, channels.10010: { $exists: false }, $and: [ { countrys.IN: { $exists: true } }, { $or: [ { category: /.*530ed96ee4b0ce13ef9146f6.*/i }, { category: /.*530ed94be4b0ce13ef9146f2.*/i } ] } ] }, $orderby: { globalRankScale: -1 }

# 看看各筛选条件记录分布
> db.themes.count({'status': { $in: [ 2, 3 ] }, 'isShared': { $ne: 0 }})
44336
> db.themes.count({'status': { $in: [ 2, 3 ] }})
44393
> db.themes.count({'status': { $in: [ 2, 3 ] }, 'themeId': { $lt: 365746 } })
6438
> db.themes.count({'themeId': { $lt: 365746 } })
336185

# 查询条件的分布
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep -v "IXSCAN { status" -c 
0
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 -c
14292
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep status  -c
14292
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep tag  -c
14292
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep -w  tag  -c
3808
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep  countrys  -c
143
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep  category   -c
4127
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep  themeId   -c
14292
wpzhou@150501d113:~/test/mongo_log$ grep anthcraft.themes mongod.log0 | grep  recommendTime  -c
3806

# 基本每个查询都涉及到status、themeId; 而且按记录分布来说，status、themeId建索引比较有用

# 但是遍历log，发现基本所有的慢查询用的都是status 的索引
# 有两种，设计到category的理论上应该走category_1_status_1_isShared_1， 但是走status； 不过按记录条数分布，走这两个索引都差不多，遍历的记录条数都为 db.themes.count({'status': { $in: [ 2, 3 ] }}) 
# 未设计category的，可能走status 或者themeId；从记录条数分布看，status更优

# 综上，建立新索引 {'status':1, 'themeId': -1} themeId 被用来排序比较多$orderby: { themeId: -1 }
db.themes.createIndex({'status':1, 'themeId': -1}, { background: true }) 
--noIndexBuildRetry

# 查看当前正在建索引的操作
db.currentOp(
                {
                  $or: [
                    { op: "command", "query.createIndexes": { $exists: true } },
                    { op: "insert", ns: /\.system\.indexes\b/ }
                  ]
                }
            )
```

