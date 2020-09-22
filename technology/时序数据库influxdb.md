# 一、Influxdb架构和简介
![图片](https://uploader.shimo.im/f/kD74Jb6EIFEbuyxd.png!thumbnail)

**1、极简架构：**单机版的InfluxDB只需要安装一个binary，即可运行使用，完全没有任何的外部依赖。

**2、极强的写入能力： **底层采用自研的TSM存储引擎，TSM也是基于LSM的思想，提供极强的写能力以及高压缩率。

**3、高效查询：**对Tags会进行索引，提供高效的检索。

**4、InfluxQL**：提供SQL-Like的查询语言，极大的方便了使用，数据库在易用性上演进的终极目标都是提供Query Language。

**5、Continuous Queries**: 通过CQ能够支持auto-rollup和pre-aggregation，对常见的查询操作可以通过CQ来预计算加速查询。

# 二、Influxdb概念解释
解释直通车：[https://www.cnblogs.com/yinfutao/p/11618892.html](https://www.cnblogs.com/yinfutao/p/11618892.html)

**Retention Policy** ：保持策略

**核心作用有3个**：指定数据的过期时间，指定数据副本数量以及指定ShardGroup Duration

1、 RP是数据库级别而不是表级别的属性。这和很多数据库都不同。

2、每个数据库可以有多个数据保留策略，但只能有一个默认策略。

3、不同表可以根据保留策略规划在写入数据的时候指定RP进行写入。

**Shard Group**：时间分区（逻辑概念）

从字面意思来看Shard Group会包含多个Shard，每个Shard Group只存储指定时间段的数据，不同Shard Group对应的时间段不会重合。

**Shard**：区

Shard Group里面包含了大量Shard，Shard才是InfluxDB中真正存储数据以及提供读写服务的概念，是InfluxDB的存储引擎实现，具体称之为TSM(Time Sort Merge Tree) Engine，负责数据的编码存储、读写服务等。

**简单总结两种分区策略**：

分布式数据库一般有两种Sharding策略：Range Sharding和Hash Sharding，前者对于基于主键的范围扫描比较高效，HBase以及TiDB都采用的这种Sharding策略；后者对于离散大规模写入以及随即读取相对比较友好，通常最简单的Hash策略是采用取模法，但取模法有个很大的弊病就是取模基础需要固定，一旦变化就需要数据重分布，当然可以采用更加复杂的一致性Hash策略来缓解数据重分布影响。

单机InfluxDB只有第一层sharding，即只有根据时间进行Range Sharding，并没有执行Hash Sharding。Hash Sharding只会在分布式InfluxDB中才会用到。

附哈希分片策略参考地址：[https://blog.csdn.net/double2hao/article/details/88760443](https://blog.csdn.net/double2hao/article/details/88760443)

# 三、TSM树存储模型
![图片](https://uploader.shimo.im/f/VQmRZVwgkzELOJMG.png!thumbnail)

1、首先将数据追加写入WAL再写入Cache就可以返回给用户写入成功，WAL可以保证即使发生异常宕机也可以恢复出来Cache中丢失的数据。一旦满足特定条件系统会将Cache中的时序数据执行flush操作落盘形成文件。文件数量超过一定阈值系统会将这些文件合并形成一个大文件。

2、在InfluxDB中，表中Tags组合会被作为记录的主键，因此主键并不唯一。

3、所有时序查询最终都会基于主键查询之后再经过时间戳过滤完成。

4、seriesKey实际上就是measurement+datasource(tags)

5、InfluxDB内部实现了倒排索引机制，即实现了tag到SeriesKey的映射关系，如果用户想根据某个tag查找的话，首先根据tag在倒排索引中找到对应的SeriesKey，再根据SeriesKey定位具体的时间线数据

![图片](https://uploader.shimo.im/f/5AB0Myr3RkEHddax.png!thumbnail)

# 四、Influxdb数据结构和存储原理
内存中实际上就是一个Map：<SeriesKey+fieldKey, List<Timestamp|Value>>，Map中一个SeriesKey+fieldKey对应一个List，List中存储时间线数据。

数据进来之后根据measurement+datasource(tags)拼成SeriesKey，加上fieldKey，再将Timestamp|Value组合值写入时间线数据List中。内存中的数据flush的文件后，同样会将同一个SeriesKey中的时间线数据写入同一个Block块内，即一个Block块内的数据都属于同一个数据源下的同一个field。

优点一：同一数据源的tags不再冗余存储。

优点二：时间序列和value可以在同一个Block内分开独立存储，独立存储就可以对时间列以及数值列分别进行压缩。

优点三：对于给定数据源以及时间范围的数据查找，可以非常高效的进行查找。

# 五、TSM存储引擎之数据写入和删除
**流程：**

1、批量时序数据shard路由：InfluxDB首先会将这些数据根据shard的不同分成不同的分组，每个分组的时序数据会发送到对应的shard。每个shard相当于HBase中region的概念，是InfluxDB中处理用户读写请求的单机引擎。

2、倒排索引引擎构建倒排索引：InfluxDB中shard由两个LSM引擎构成 – 倒排索引引擎和TSM引擎。时序数据首先会经过倒排索引引擎构建倒排索引，倒排索引用来实现InfluxDB的多维查询。

3、TSM引擎持久化时序数据：倒排索引构建成功之后时序数据会进入TSM Engine处理。TMS Engine处理流程和通用LSM Engine基本一样，先将写入请求追加写入WAL日志，再写入cache，一旦满足特定条件会将cache中的时序数据执行flush操作落盘形成TSM File。

**倒排索引构建：**

1、拼出来SeriesKey

2、确认SeriesKey是否已经构建过索引了

3、如果seriesKey在文件中不存在，需要将其写入内存。倒排索引内存结构主要包含两个Map：<measurement, List<tagKey>> 和 <tagKey, <tagValue, List<SeriesKey>>>

**数据删除操作：**

![图片](https://uploader.shimo.im/f/n8ZBtV1Ui08aom6G.png!thumbnail)

**同步删除策略：**

1、删除所有TSM File中满足条件的series

2、删除Cache中满足条件的series

3、在WAL中生成一条删除series的记录并持久化到硬盘

**标记tag删除策略：**

1、在WAL中生成一条flag为deleted的LogEntry并持久化到硬盘。

2、将要删除的维度信息写入Cache，需要标记deleted（设置type=deleted）。

3、当WAL大小超过阈值之后标记为deleted的维度信息会随Cache Flush到倒排索引文件。

4、Inverted Index Engine中索引信息真正被删除发生在compact阶段。

# 六、Influxdb读取流程
![图片](https://uploader.shimo.im/f/GDxuwd9ZQpg7yTPY.png!thumbnail)

1.、Query：InfluxQL允许用户使用类SQL语句执行查询分析聚合

2、InfluxQL进入系统之后，系统首先会对InfluxQL执行切词并解析为抽象语法树（AST），抽象树中标示出了数据源、查询条件、查询列以及聚合函数等等。

3、BuildIterators：InfluxQL语句转换为Query实体对象之后，就进入读取流程中最重要最核心的一个环节 – 构建Iterator体系。

查询按照查询列构建最顶层FieldIterator，每个FieldIterator会根据TimeRange雇佣多个ShardIterator去处理单个Shard上面对应列值的查找，对查找到的值要么直接返回要么执行Reduce函数进行聚合操作。每个Shard内部首先会根据查询条件利用倒排索引定位到所有满足条件的series，再为每个series构建一个TagsetIterator用来查找具体的列值数据。因此，TagsetIterator是整个体系中唯一干活的Iterator，所有其他上层Iterator都是逻辑Iterator。

![图片](https://uploader.shimo.im/f/WLWxZlTdJJ4P6lQb.png!thumbnail)

1. 根据where condition以及所有倒排索引文件查处所有满足条件的SeriesKey

2. 将满足条件的SeriesKey根据GroupBy维度列进行分组，不同分组后续的所有操作都可以独立并发执行，因此可以多线程处理

3. 针对某个分组的SeriesKey集合以及待查询列，根据指定查询时间段（TimeRange）在所有TSMFile中根据B+树索引构建查询iterator

4. 将满足条件的原始数据返回给上层进行聚合运算，并将聚合运算的结果返回给用户

# 七、influxdb安装使用
**1、安装**

sudo curl -sL [https://repos.influxdata.com/influxdb.key](https://repos.influxdata.com/influxdb.key) | sudo apt-key add -

sudo apt-get update

sudo apt-get install influxdb

sudo systemctl start influxdb

sudo  systemctl enable influxdb

sudo systemctl is-enabled influxdb

**2、注意点**

1、Tag
    只能使用Tag进行Group

    只能使用Tag进行正则表达式操作

2、Field
    只能使用Field进行函数操作，例如sum()

    只能使用Field进行比较操作
    
    如果需要使用int,float,boolean类型进行存储,只能使用Field

**3、优化建议**

1.把相似的数据放到相同的measurement里，容易被压缩和查询。

2.分离不相似的数据到不同的数据库，在一个给定的时间内，只有一个程序负责把数据写入到一个shard里，因此越能将shard分离出来(拆分为不同的数据库)，同时发生的写入就越多。

3.不应该选择比收集间隔(collection interval)更小的精度，否则会浪费空间和导致更大的性能开销

4.有规律的间隔点会带来更高效的压缩。(如每隔10秒)。

**4、数据备份和回复**

**备份：**

influxd backup

    [ -database <db_name> ]  --> 指定需要备份的数据库名
    
    [ -portable ]            --> 表示在线备份
    
    [ -host <host:port> ]    --> influxdb服务所在的机器，端口号默认为8088
    
    [ -retention <rp_name> ] | [ -shard <shard_ID> -retention <rp_name> ]  --> 备份的保留策略，注意shard是挂在rp下的；我们需要备份的就是shard中的数据
    
    [ -start <timestamp> [ -end <timestamp> ] | -since <timestamp> ]   --> 备份指定时间段的数据
    
    <path-to-backup>   --> 备份文件的输出地址

**例子：**

备份所有：influxd backup -portable /tmp/data/total

备份test库：influxd backup -portable -database test /tmp/data/test

备份指定时间段：influxd backup -portable -database test -start 2018-07-27T2:31:57Z -end 2018-07-27T2:32:59Z  /tmp/data/test_per

**恢复：**

influxd restore 

    [ -db <db_name> ]       --> 待恢复的数据库(备份中的数据库名)
    
    -portable | -online
    
    [ -host <host:port> ]    --> influxdb 的服务器
    
    [ -newdb <newdb_name> ]  --> 恢复到influxdb中的数据库名
    
    [ -rp <rp_name> ]        --> 备份中的保留策略
    
    [ -newrp <newrp_name> ]  --> 恢复的保留策略
    
    [ -shard <shard_ID> ]
    
    <path-to-backup-files>

**例子：**

备份到不存在的：influxd restore -portable -db yhhblog -newdb yhhblog_bk yhhblog_per

# 八、influxdb常用场景
**1、查询函数（类sql）**

select zipcode,val,usex from mockbigdata where ad='iphone' and adGroup='game' and app='bigman' limit 1000;

注：where 后面应该是设置好的tag，tag多查询速度快。尽量不要select *，只查询所需要的字段。

**2、变换类函数**

DERIVATIVE：

InfluxDB会计算按照时间进行排序的字段值之间的差异，并将这些结果转化为单位变化率。其中，单位（unit）可以指定，默认为1s。

SELECT DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]

DIFFERENCE：

返回一个字段中连续的时间值之间的差异。字段类型必须是长整型或float64。

SELECT DIFFERENCE(<field_key>) FROM <measurement_name> [WHERE <stuff>]

MOVING_AVERAGE：

返回一个连续字段值的移动平均值，字段类型必须是长整形或者float64类型。

SELECT MOVING_AVERAGE(<field_key>,<window>) FROM <measurement_name> [WHERE <stuff>]

# 九、influxdb分布式拓展建议
除了购买企业版以外，目前有两个自研开源的解决方案：

1、360的解决方案：[https://github.com/Qihoo360/influx-proxy](https://github.com/Qihoo360/influx-proxy)

2、饿了么的解决方案：[https://github.com/shell909090/influx-proxy](https://github.com/shell909090/influx-proxy)

简单拓展方法可以写一个中心节点的代理服务，转发agent的操作请求，分发到不同的业务节点处理。如果有较多的热点数据，则需要考虑负载均衡。

![图片](https://uploader.shimo.im/f/mKDeowGFQfs5w9sn.png!thumbnail)

1、监控监测报警可以直接接入钉钉

2、添加配置后，可以将influxdb地址注册到http-proxy中，根据配置文件的measurements，将query和write请求映射到指定的机器处理。配置文件支持热更新。

3、dash monitor可以支持，停止，启动和更新任务，添加节点删除节点，measurements热点数据动态分配存储。查看任务执行状况，查看数据仓库吞吐量和机器运行的cpu、内存等主要指标。

# 十、使用风险项
1、新版中influxdb已经取消了web管理台，新版本的HTTP接口与老版本query参数不一样，需要关注接口版本。

2、提取数据需要考虑返回的数据量，避免过大crash。

3、热点数据分散处理策略，需要考虑到查询场景，不能影响查询性能。







