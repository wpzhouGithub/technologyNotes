



## GIS的主要使用场景

- GIS智慧城市
- GIS大数据可视
- GIS的应用领域很多，比如自然资源部门、气象部门、一些大型的互联网公司等等



## GIS数据枚举

- 矢量数据

- 影像数据

- 三位数据

- 视频数据

- 倾斜摄影模型

- BIM模型

- 

- 手机信令数据

- 移动社交数据

- 导航终端数据

- 车船位置信息

- 社交媒体数据

- 电商交易数据

- 导航轨迹数据

- 搜索关键词数据

- 公交地铁刷卡数据

  

## GIS整体架构

- 开源平台和商业平台（底层的server和三方产品的server）
- 开源和商业的API库（底层的web的框架和三方产品的web的框架）
- web 调用API库，使用web的地图相关的插件显示



## OGC标准

- 该标准规定了GIS相关的server和web之间交互，需要再仔细关注。
- 特别是谁渲染的问题。WFS、WMF、WMFS，具体使用哪个，需要根据具体的使用场景
  - 数据量少的，直接使用WFS
  - 需要固定绘图的，WMF
  - 数据量大，需要使用瓦片，WMFS
- [ ] 需要再仔细关注



## 成熟产品学习：了解ArcGIS、超图、中地数码

### ArcGIS

- 由美国环境系统研究所（ESRI）开发的， 用于创建和共享交互式 web 地图的基于云的软件

- 接入自己的位置数据，对数据做指定处理之后，通过API返回； 

- web请求API获取数据，使用ArcGIS提供的web插件实现各种地图效果。

  - 用户可以直接在web 地图上交互，比如拖拽、放大、搜索等
  - 地图效果让数据更直观，展现形式更多样，方便用户能够获取更多有效的信息
  - 形成的web效果，方便分享、传播
  - 通过协作构建和使用地图，在整个组织中有效工作。主要是指安全性方面的控制，如用户管理、权限管理

- 功能

  - **制作地图**：在地图上展示数据，可以使用arcgis的权威数据对自己的数据做补充，从而呈现更好的表现效果。在地图上可以额外显示一些数据信息

    ![img](D:\WorkSpace\technologyNotes\pictures\GIS随笔\agol-overview-tcs-make-maps.gif)
  
    - **具体的使用还需要详细看**？？？？？？？？？？？？？
  - 共享地图和应用程序：在web中构建地图效果，如网站、社交媒体文章和博客文章等。web分享传播更方便。
  
    - **权限管理**
    - 提供相关功能，**能够快捷的创建web**。如dashboard。即指定数据源，能够通过拖拽直接生成各种数据展示效果。
  
  - 协作：主要是指**安全性方面**的控制，如用户管理、数据访问权限管理
  
    - 自定义群组
    - 用户类型/角色
  
  - **分析数据**：通过使用直观的分析工具了解位置环境中的数据
    
    - **具体的使用还需要详细看**？？？？？？？？？？？？？
    
  - 使用数据：可对数据进行托管、缩放扩展，同时可以进行地理启用（使用arcgis自有的地理数据，对用户的数据进行补充，比如行政区域信息等）。会有权限的控制。在新增数据时，不会影响已生成的web展现（展现的数据应该是通过计算，存储的另外的数据，当然不会有影响）
  
    - 可以将以下来源的数据导入到arcgis：表格、地理空间文件、图片、某些数据库服务
  
    - 保持数据的最新状态：可设置定时更新，将用户自己的数据更新到arcgis
  
    - 托管基础架构？？？？？？？？？？？？？
  
    - 精确采集所需信息：ArcGIS Online 可以帮助您有效采集、处理和使用数据（设置参数，控制采集结果）
  
    - **精确控制数据访问：通过为相同数据集创建独特**视图**来控制可以使用数据的人员**（下至字段和记录级别）。 各个视图可以赋予每个受众相应的数据权限，因此无需管理多个数据集。
      
    - **使用arcgis自有的数据对用户的数据做补充**：
      - 添加位置：在 ArcGIS Online 中使用位置数据（例如行政边界或 World Geocoding Service）在地图上绘制表格数据。
      
      - 识别模式： 按照预定义的地理位置（例如邮政编码）汇总数据，以计算平均值和检测模式。
      
      - 丰富数据：通过使用人口统计和生活方式信息丰富数据来了解您绘制的位置周围的人口、住房和商业。
      
    - 在其他系统中使用 ArcGIS Online 数据：从arcgis讲数据导出，然后在其他的系统使用。如OGC规定的格式、社区标准格式（如GeoJSON）
    
  - 
  

### 超图学习

- 基础软件

  - 底层大数据架构：HBase、HDFS、Elasticsearch、Kafka、spark

  - 基于大数据做数据分析：包括人工智能、三维GIS、分布式GIS

  - 跨平台主要是指能在众多平台上使用，用来兼容不同的平台、系统

  - 边缘GIS：资源分发缓存、数据本地处理，主要是为了提高客户端资源请求响应、降低云GIS的压力
  - 区块链服务的使用场景？？？？？？？？？？？？？？？？？？？？？？

- 超图大数据GIS白皮书学习
  - 需要解决的痛点：空间大数据存储、计算、流计算、可视化
  - 空间大数据的特点：体量大、种类多、变化快、价值密度低
  - 云原生GIS：其实质是将GIS的平台、软件和地理空间信息方便、高效的部署到以云计算为支撑的“云”基础设施之上，以弹性、按需的方式提供服务
  - 空间大数据存储使用的技术： 
    - 基于 Hadoop 的HDFS 实现非结构化数据存储； 
    - 通过对 Postgres-XL 分布式数据库的支持对海量空间大数据提供存储管理； 
    - 通过对 MongoDB 分布式数据库的支持对海量二维或者三维的瓦片数据提供存储管理；
    -  通过对 Elasticsearch 分布式数据库的支持对流数据提供存储管理等； 
    - 基于分布式架构的 HBase 是分布式空间数据存储和管理的首选；
    -  通过对 DSF 分布式空间文件引擎(DSF，Distributed Spatial File) 支持实现了高性能全量计算能力  
    -  DSF 和HDFS对比，具体增加了空间相关的什么内容？？？？？？？？？？？
    - 具体的使用、理解各种类型数据对存储形式的选型原因，后面在实践阶段重点关注？？？？？？？？？？？？？
  - 超图大数据GIS技术
    - 使用分布式文件系统、分布式数据库实现高效稳定的存储
    - 提供了 **SuperMap iObjects for Spark **空间大数据组件， 从内核扩展了 Spark
      空间数据模型，  使用Spark实现了各种空间分析算法

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata.png)

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata1.png)

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata2.png)

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata3.png)

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata4.png)

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata5.png)

![](D:\WorkSpace\technologyNotes\pictures\GIS随笔\supermap-bigdata6.png)





- 阅读超图的大数据相关的使用文档，没什么收获。hadoop \spar\mongodb等都是分开安装，然后在iserver里面配置的。具体的实现细节，没什么发现。



## GIS理论学习



## postgis学习

- 安装：window10安装
  - 参考：https://www.cnblogs.com/haolb123/p/14330532.html
- shapefile 存储到postgis
  - python
  - 工具
- postgis中tiger、tiger_data....数据库的作用
- ## **PostGIS是什么？**

  PostGIS通过向PostgreSQL添加对**空间数据类型**、**空间索引**和**空间函数**的支持，将PostgreSQL数据库管理系统转换为**空间数据库**。

  因为PostGIS是建立在PostgreSQL之上的，所以PostGIS自动继承了重要的"**企业级**"特性以及开放源代码的标准。

  可以说PostGIS仅仅只是PostgreSQL的一个插件，但是它将PostgreSQL变成了一个强大的空间数据库！
- 在查询之前需要加载PostGIS空间扩展，在查询文本区域中输入以下查询语句以加载**PostGIS空间扩展**：

  ```sql
  CREATE EXTENSION postgis;
  ```



- PG服务搭建存储，存储矢量数据
  - 先直接在现有的单机的测试机上完成矢量数据的存储
  - 先实现单机的理论到实践走通，然后再尝试集群模式
- postgis是在postgreSQL基础做做了关于空间的扩展； porstgres-XL是对postgreSQL的集群支持，实现高并发读写、事务一致性； 后期大数据时，可以使用 porstgres-XL+postgis
- 矢量数据、存储模型
- 数据引擎



## hbase 对矢量数据的存储

- 读了两篇相关的论文，都主要涉及到存储模型的设计。模型需要cover到后面主要的查询场景。
- hbase 是通过rowkey去快速定位数据的，所以要特别关注rowkey的设计。
- 两篇文章里面的模型并不一样，不过具体的数据、查询场景不一样。
- 综上，要在hbase里面存数据，需要针对不同的数据、不同的查询场景个性化设计。需要对数据、查询场景做详细的研究、了解，GIS理论就显得很重要了。



## 瓦片地图原理

**瓦片**
 指将一定范围内的地图按照一定的尺寸和格式，按缩放级别或者比例尺，切成若干行和列的正方形栅格图片，对切片后的正方形栅格图片被形象的称为瓦片（Tile）。

![img](https:////upload-images.jianshu.io/upload_images/5017748-4cdebce16e47b7d0.png?imageMogr2/auto-orient/strip|imageView2/2/w/323/format/webp)

瓦片.png

![img](https:////upload-images.jianshu.io/upload_images/5017748-9f2a75cb11db2efc.png?imageMogr2/auto-orient/strip|imageView2/2/w/436/format/webp)

地图瓦片.png

**层次模型**
 瓦片地图金字塔模型是一种多分辨率层次模型，从瓦片金字塔的底层到顶层，分辨率越来越低，但表示的地理范围不变。

![img](https:////upload-images.jianshu.io/upload_images/5017748-5b9823484464641a.png?imageMogr2/auto-orient/strip|imageView2/2/w/723/format/webp)

地图瓦片金字塔模型.png

![img](https:////upload-images.jianshu.io/upload_images/5017748-e2dc27daef44ac7f.png?imageMogr2/auto-orient/strip|imageView2/2/w/845/format/webp)

- 切瓦片

![img](D:\WorkSpace\technologyNotes\pictures\GIS随笔\v2-f9bdb0903e0e654609f712bd49ead771_720w.jpg)





## 瓦片的存储

- 瓦片的生成， 当前系统的实现

  - 在geoserver框架中配置，生成瓦片。
  - geowebcache执行切片
  - 实现geowebcache的输出接口，将瓦片直接存mongodb
  - 具体的存储模型需要着重关注

- mongodb 瓦片存储模型

  - 超图、arcgis、中地的实现
    - 安装超图，将瓦片到处到mongodb，分析数据结构
    - 安装mongodb cluster模式，到处数据看看
    - 安装中地看看存储格式
    - 
  - 论文
  - tile to mongodb 的文章
    - gridfs-locks
    - MongoDB's built-in GridFS 

  ```javascript
  var mongoose = require('mongoose')
  
  
  var TileSchema = new mongoose.Schema({
    zoom_level: Number,
    tile_column: Number,
    tile_row: Number,
    tile_data: Buffer
  })
  
  
  var TilesetSchema = new mongoose.Schema({
    tileset_id: { type: String, index: true },
    owner: String,
  
    // tilejson spec
    tilejson: { type: String, default: '2.1.0' },
    name: String,
    description: String,
    version: { type: String, default: '1.0.0' },
    attribution: String,
    template: String,
    legend: String,
    scheme: { type: String, default: 'xyz' },
    tiles: [String],
    grids: [String],
    data: [String],
    minzoom: { type: Number, default: 0 },
    maxzoom: { type: Number, default: 22 },
    bounds: { type: [Number], default: [-180, -90, 180, 90] },
    center: [Number],
  
    formatter: String,
    resolution: Number,
    format: String,
    vector_layers: [{
      _id: false,
      id: String,
      description: String,
      minzoom: Number,
      maxzoom: Number,
      source: String,
      source_name: String,
      fields: mongoose.Schema.Types.Mixed
    }]
  }, { timestamps: true })
  
  
  TileSchema.index({zoom_level: 1, tile_column: 1, tile_row: 1})
  
  
  module.exports.TileSchema = TileSchema
  module.exports.TilesetSchema = TilesetSchema
  ```

![image-20220428133424771](D:\WorkSpace\technologyNotes\pictures\GIS随笔\image-20220428133424771.png)


## 一张图看懂三维GIS

![img](D:\WorkSpace\technologyNotes\pictures\GIS随笔\SouthEast.png)

## GPU CPU画图的性能



## 是否有开源的软件开源借鉴

