## MySQL查询过程流程图

![](../../pictures/mysql_query.jpeg)

## 查询缓存

### 查询缓存命中

- 在解析一个查询语句前，如果查询缓存是打开的，那么MySQL会检查这个查询语句是否命中查询缓存中的数据。如果当前查询恰好命中查询缓存，在检查一次用户权限后直接返回缓存中的结果。这种情况下，查询不会被解析，也不会生成执行计划，更不会执行。

- MySQL将缓存存放在一个引用表（不要理解成table，可以认为是类似于HashMap的数据结构），通过一个哈希值索引，这个哈希值通过查询本身、当前要查询的数据库、客户端协议版本号等一些可能影响结果的信息计算得来。所以**两个查询在任何字符上的不同（例如：空格、注释），都会导致缓存不会命中**。

### 不会被缓存的查询

- 如果查询中包含任何用户自定义函数、存储函数、用户变量、临时表、mysql库中的系统表，其查询结果都不会被缓存。比如函数`NOW()`或者`CURRENT_DATE()`会因为不同的查询时间，返回不同的查询结果，再比如包含`CURRENT_USER`或者`CONNECION_ID()`的查询语句会因为不同的用户而返回不同的结果，将这样的查询结果缓存起来没有任何的意义。

### 缓存何时失效

- **缓存涉及到的表（结构和数据）发生变化，缓存就会失效**；SO**任何写操作都会造成缓存失效**
- 基于此，我们要知道并不是什么情况下查询缓存都会提高系统性能，缓存和失效都会带来额外消耗，只有当缓存带来的资源节约大于其本身消耗的资源时，才会给系统带来性能提升。
- 当前的使用场景中写多查少，可以尝试关闭缓存、优化读写SQL
- 

