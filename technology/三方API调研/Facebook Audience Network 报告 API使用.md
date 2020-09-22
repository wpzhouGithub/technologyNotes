@[TOC](Facebook Audience Network 报告 API使用)

## Token 的获取、验证

1. token有两种：**短期用户口令**， **系统用户口令**
2. 短期用户口令容易过期，使用系统用户口令； 直接在控制台生成即可
3. token使用比较简单，相比AdSense 不用再验证，直接作为参数传入即可



## 调用API

一共有3中API调用：同步生成报告、异步生成报告、使用query_id获取报告

所有数据均以 GMT - 8 时区返回。

### 1. 同步生成报告

```shell
# GET请求
https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID|PROPERTY_ID>/adnetworkanalytics/?
  metrics=['fb_ad_network_imp']&
  access_token=<ACCESS_TOKEN> 
  
```

**最多只能请求1个指标、2个细分数据，不能加筛选条件**

### 2. 异步生成报告，返回query_id

```shell
# POST请求
https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID|PROPERTY_ID>/adnetworkanalytics/?
  metrics=['fb_ad_network_imp', 'fb_ad_network_request']&
  filters=[{'field':'country', 'operator':'in', 'values':['US']}]&
  breakdowns=['platform']&
  access_token=<ACCESS_TOKEN> 
```

相比同步生成报告：

- 请求方式是POST

- 参数相同，除了个数条件限制

- 异步旨在处理较重的数据负载，它会先返回一个**查询编号**，供您稍后检索数据。下面将提供更多信息。**请注意：**您**可以**在 **12 个小时**内使用**查询编号**的结果



### 3. 使用query_id获取报告

```shell
# GET请求
https://graph.facebook.com/<API_VERSION>/<BUSINESS_ID|PROPERTY_ID>/adnetworkanalytics_results/?
  query_ids=['QUERY_ID']&
  access_token=<ACCESS_TOKEN> 
  
```

请求的链接为adnetworkanalytics_results



## 指标

| 名称                                    | 说明                                                         | 语法                                                |
| --------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| `fb_ad_network_revenue`                 | 预计收入                                                     | `metrics=['fb_ad_network_revenue']`                 |
| `fb_ad_network_request`                 | 广告请求数量                                                 | `metrics=['fb_ad_network_request']`                 |
| `fb_ad_network_cpm`                     | 预计千次展示费用                                             | `metrics=['fb_ad_network_cpm']`                     |
| `fb_ad_network_click`                   | 点击量                                                       | `metrics=['fb_ad_network_click']`                   |
| `fb_ad_network_imp`                     | 展示次数                                                     | `metrics=['fb_ad_network_imp']`                     |
| `fb_ad_network_filled_request`          | 填充的广告请求数量                                           | `metrics=['fb_ad_network_filled_request']`          |
| `fb_ad_network_fill_rate`               | 广告请求填充率                                               | `metrics=['fb_ad_network_fill_rate']`               |
| `fb_ad_network_ctr`                     | 预计点击率                                                   | `metrics=['fb_ad_network_ctr']`                     |
| `fb_ad_network_show_rate`               | 展示数除以填充请求数                                         | `metrics=['fb_ad_network_show_rate']`               |
| `fb_ad_network_video_guarantee_revenue` | 视频保证收入                                                 | `metrics=['fb_ad_network_video_guarantee_revenue']` |
| `fb_ad_network_video_view`              | 10 秒视频观看量                                              | `metrics=['fb_ad_network_video_view']`              |
| `fb_ad_network_video_view_rate`         | 0 秒视频观看率                                               | `metrics=['fb_ad_network_video_view_rate']`         |
| `fb_ad_network_video_mrc`               | 2 秒视频观看量                                               | `metrics=['fb_ad_network_video_mrc']`               |
| `fb_ad_network_video_mrc_rate`          | 2 秒视频观看率                                               | `metrics=['fb_ad_network_video_mrc_rate']`          |
| `fb_ad_network_bidding_request`         | 竞价请求数量                                                 | `metrics=['fb_ad_network_bidding_request']`         |
| `fb_ad_network_bidding_response`        | 竞价响应数量                                                 | `metrics=['fb_ad_network_bidding_response']`        |
| `fb_ad_network_bidding_bid_rate`        | 竞价响应率                                                   | `metrics=['fb_ad_network_bidding_bid_rate']`        |
| `fb_ad_network_bidding_win_rate`        | 竞价工具赢得竞拍的比率                                       | `metrics=['fb_ad_network_bidding_win_rate']`        |
| `fb_ad_network_no_fill`                 | 无填充主因数量 **仅适用于用作单个指标 fail_reason 细分数据的情况** | `metrics=['fb_ad_network_no_fill']`                 |
| `fb_ad_network_no_bid`                  | 未竞价主因数量 **仅适用于用作单个指标 fail_reason 细分数据的情况** | `metrics=['fb_ad_network_no_bid']`                  |

## 可选参数

### 1. 细分数据

**同步**请求支持每个请求**两个**细分数据。
**异步**请求对细分数据的数量未作限制，但包含的细分数据越多，生成数据所需的时间越长。请求太多指标和细分数据可能会导致结果丢失。

**不用用作筛选**

| 名称              | 说明                                                         | 语法                             |
| ----------------- | ------------------------------------------------------------ | -------------------------------- |
| `country`         | 按指标发生的国家/地区来区分的细分数据。                      | `breakdowns=['country']`         |
| `delivery_method` | 当指标来自通过 AN 竞价投放的广告时，按*标准*或*竞价*区分的细分数据。 **仅适用于使用创收管理工具的发行商** | `breakdowns=['delivery_method']` |
| `platform`        | 按记录指标的平台细分。可以是 `ios`（移动应用）、`android`（移动应用）、`mobile_web` 或 `instant_articles`。 **仅适用于使用创收管理工具的发行商** | `breakdowns=['platform']`        |
| `property`        | 按资产编号细分。                                             | `breakdowns=['property']`        |
| `placement`       | 按版位编号细分。                                             | `breakdowns=['placement']`       |
| `placement_name`  | 按版位编号和版位名称分类的细分数据                           | `breakdowns=['placement_name']`  |
| `fail_reason`     | 按失败原因分类的细分数据。 **仅适用于 fb_ad_network_no_fill 和 fb_ad_network_no_bid 指标** | `breakdowns=['fail_reason']`     |

### 2. 自/至

如果未包含，默认为过去 7 天。

您现在可以将 Unix 时间戳用于自/至值！您必须至少选择 1 小时来与 Unix 时间戳配合使用。

同步端点（GET 请求）支持 **7 天**。
异步端点（POST 请求）支持 **30 天**。

**示例代码中，不管是同步还是异步，都限定的最大范围为7天； 可以改一下参数验证的代码**

**日期会被以用户设置的时区转换为时间戳**

| 名称    | 说明           | 语法                                           |
| ------- | -------------- | ---------------------------------------------- |
| `since` | 查询的起始限制 | `since=2018-12-01` 或 `since=1548880485`       |
| `until` | 查询的结束限制 | `until=2019-01-01` 或 `until=1548880485+86400` |

### 3. 筛选条件

**细分数据为受支持的筛选条件**。它们必须包括 `field`、`operator` 和 `values`。**当前仅支持 `in` 运算符**。

| 名称              | 说明                                                         | 语法                                                         |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `country`         | 按指标发生的国家/地区筛选。                                  | `filters=[{'field':'country', 'operator':'in', 'values':['US', 'JP']}]` |
| `placement`       | 按版位编号筛选。如果小于 100，值为 `REDACTED`。              | `filters=[{'field':'placement', 'operator':'in', 'values':['PLACEMENT_ID']}]` |
| `delivery_method` | 按 `standard` 或 `bidding` 筛选。                            | `filters=[{'field':'delivery_method', 'operator':'in', 'values':['bidding']}]` |
| `platform`        | 按记录指标的平台筛选。可以是 `ios`（移动应用）、`android`（移动应用）、`mobile_web` 或 `instant_articles`。 | `filters=[{'field':'platform', 'operator':'in', 'values':['mobile_web']}]` |

### 4. 栏排序

如果未包含，**默认为 `time`**。

| 名称    | 说明                   | 语法                    |
| ------- | ---------------------- | ----------------------- |
| `time`  | 按每行显示的时间戳排序 | `ordering_column=time`  |
| `value` | 按每行的值排序         | `ordering_column=value` |

### 5. 排序类型

如果未包含，默认为 `descending`。

| 名称         | 说明         | 语法                       |
| ------------ | ------------ | -------------------------- |
| `ascending`  | 从小到大排序 | `ordering_type=ascending`  |
| `descending` | 从大到小排序 | `ordering_type=descending` |

### 6. 限制

| 名称    | 说明                                                         | 语法        |
| ------- | ------------------------------------------------------------ | ----------- |
| `limit` | 限制响应行数的整数。**同步请求的最大行数限制为 2,000，异步请求的最大行数限制为 20,000** | `limit=500` |

**参数验证中，分别现在的1000、10000； 可改代码，扩大限制**

### 7. 汇总周期

如果未包含，默认为 `day`。

现在您可以按小时汇总数据， 但是请求周期不能超过 2 天； 超过的就不能获得每小时的汇总。

**所以，请求的日期要大于2天**

| 名称    | 说明                     | 语法                       |
| ------- | ------------------------ | -------------------------- |
| `hour`  | 按小时汇总数据           | `aggregation_period=hour`  |
| `day`   | 按天汇总数据             | `aggregation_period=day`   |
| `total` | 汇总整个时间范围内的数据 | `aggregation_period=total` |

## 8. 实时获取数据

- 当前无法获取到实时的收入数据，最早可以获取到最近2小时之前的每个整小时的数据；
- Facebook控制台显示的小时的时区是utc-8;  按天显示的日期时区是用户设置的时区，当前是utc-8;
- 汇总周期为小时（aggregation_period=hour）时， since、until 设置的时间范围必须是最近48小时以内；
-  since、until传参：如果传入的是日期，都会被当做utc+8的日期；传入时间戳时，返回的数据是指定时间戳范围的数据；
- 返回的数据时间都是utc时间
- 比如1点的小时数据是1点到2点产生的



## 疑点

1. 时区的设置： 接口所有数据均以 GMT - 8 时区返回。