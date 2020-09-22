

 @[TOC](Google AdSense API 获取报告)



## 一. 整体账号情况

1. 目前有3个不同的账号（accountId），每个账号下分应用（AD_CLIENT_ID）
2. 不同账号下的应用有交集，在获取数据之后，需要将相同应用的数据归类； 就需要维护一个AD_CLIENT_ID 与应用的映射表
3. 各个账号需要放在不同的IP下运行



## 二. token 的验证

1. console 生成json 格式的token，第一次认证的时候，需要手动打开返回的url，同意一系列的用户政策之后，认证通过

   - token有两种：web和installed

   最初用的web 的，但是打开验证url 的时候，400错误

   installed 的，打开返回的url，同意一系列的用户政策之后，认证通过 

2. 认证之后，会在当前目录生成adsense.dat，下次验证的时候，直接读取该文件
3. credit_path的设置，因为有多个账号token的存在，想把各个账号用文件夹分开，所以把client_secrets.json、adsense.dat都放到了各账号的文件夹下
```python
service, flags = sample_tools.init(
    ['--noauth_local_webserver'], 'adsense', 'v1.4', __doc__, credit_path,
    scope='https://www.googleapis.com/auth/adsense.readonly')

```

4. 问题来了

   - warning : adsense.dat 找不到

   - 提示url，需要重新手动验证

   - 猜测应该是adsense.dat 读取失败导致的，可是adsense.dat 明明是放在credit_path目录下的

   - **读源码得知，只会在当前目录查找adsense.dat；即并没有去credit_path指定的文件夹读取**

   - 在运行代码之前，直接cd credit_path； 这样就万事大吉了




## 三. 相关概念

### 指标和维度的说明及兼容性

https://developers.google.com/adsense/management/metrics-dimensions



### 接口的使用

调用reports 有两种形式

#### 1. Accounts.reports: generate

直接传入相关的参数，输出报告，直接解析数据

```python

      result = service.accounts().reports().generate(
          accountId=account_id, startDate='today-2m', endDate='today',
          metric=['PAGE_VIEWS', 'AD_REQUESTS', 'AD_REQUESTS_COVERAGE',
                  'CLICKS', 'AD_REQUESTS_CTR', 'COST_PER_CLICK',
                  'AD_REQUESTS_RPM', 'EARNINGS'],
          dimension=['MONTH', 'PLATFORM_TYPE_NAME'],
          sort=['+MONTH']).execute()

```



#### 2. Accounts.reports.saved

需要提前制定好报告，类似报告模板，然后再生成报表，解析数据

```python
    # Retrieve ad client list in pages and display data as we receive it.
    request = service.accounts().reports().saved().list(
        accountId=account_id, maxResults=MAX_PAGE_SIZE)
        
    # 通过 savedReportId，获取报告结果
    result = service.accounts().reports().saved().generate(
    accountId=account_id, savedReportId=saved_report_id).execute()
```



**具体的，怎么生成报告模板，暂时还不清楚； 控制台可以直接手动制定；**

**目前不需要制定报告模板，使用第1种方法，直接生成报告，解析数据**



#### 3. Accounts.reports: generate参数

| 参数名称               | 值        | 说明                                                         |
| :--------------------- | :-------- | :----------------------------------------------------------- |
| **路径参数**           |           |                                                              |
| `accountId`            | `string`  | 要为其生成报告的帐户。                                       |
| **必需查询参数**       |           |                                                              |
| `endDate`              | `string`  | 要生成报告的日期范围的结束日期（包含此结束日期），格式为“YYYY-MM-DD”。 |
| `startDate`            | `string`  | 要生成报告的日期范围的开始日期（包含此开始日期），格式为“YYYY-MM-DD”。 |
| **可选查询参数**       |           |                                                              |
| `currency`             | `string`  | 生成货币指标报告所使用的币种。如未设置，则默认为帐户的币种。 |
| `dimension`            | `string`  | 报告依据的维度。可用的维度在[指标和维度](https://developers.google.com/adsense/management/metrics-dimensions)中有所说明。要详细了解每个维度，请参阅[AdSense术语库](https://support.google.com/adsense/bin/topic.py?topic=19363)。 |
| `filter`               | `string`  | 应用到报告的[过滤器](https://developers.google.com/adsense/management/v1.2/filters.html)。 |
| `locale`               | `string`  | 用于将报告输出翻译为本地语言的语言区域。如果未指定，则默认为“en_US”。 |
| `maxResults`           | `integer` | 要返回的报告数据的行数上限。如果startIndex未设置或为零，则API将返回maxResults值指定的所有行。如果未设置maxResults，则API将返回尽可能多的行，最多可达50000行。 如果startIndex设置为非零值，则startIndex和maxResults的总数不得超过5000，否则将返回错误：不支持超过5000行的分页。可接受的值包括`0`到`50000`（含0和50000）。 |
| `metric`               | `string`  | 要包含在报告中的数字列。可用的指标在[指标和维度](https://developers.google.com/adsense/management/metrics-dimensions)中有所说明。要详细了解每个指标，请参阅[AdSense术语库](https://support.google.com/adsense/bin/topic.py?topic=19363)。 |
| `sort`                 | `string`  | 对生成的报告进行排序所使用的[维度或指标](https://developers.google.com/adsense/management/metrics-dimensions)的名称。可以选择性加上“+”前缀（按升序排序）或“-”前缀（按降序排序）。如果未指定任何前缀，则列将按升序排序。 |
| `startIndex`           | `integer` | 要返回的第一行报告数据的索引。如果startIndex未设置或为零，则API将返回maxResults值指定的所有行。如果未设置maxResults，则API将返回尽可能多的行，最多可达50000行。 如果startIndex设置为非零值，则startIndex和maxResults的总数不得超过5000，否则将返回错误：不支持超过5000行的分页。可接受的值包括`0`到`5000`（含0和5000）。 |
| `useTimezoneReporting` | `boolean` | 报告是否应该按照AdSense帐户的本地时区生成。如果为false，则会使用默认的PST/PDT时区。 |



#### 4. 参数设置

##### dimension 设置

AD_CLIENT_ID： 应用。示例：ca-pub-088712

DATE：日期

AD_UNIT_CODE：广告单元的唯一ID。示例：`837572`

AD_UNIT_NAME：广告单元的名称。

COUNTRY_CODE：CLDR地区代码，国家缩写。示例：`US`、`FR`。

#AD_FORMAT_NAME： 广告格式



##### 日期范围

endDate，startDate， 格式：“YYYY-MM-DD”； 

当返回的记录条数比较多的时候，可以减小日期范围，多请求几次，避免超过最大现在50000；



##### 报告的时区

useTimezoneReporting： True 直接使用用户设置的报告时区，目前都设置的utc+8



##### 翻页设置

startIndex 、maxResults 

如果不传startIndex ，一共最多可以获取50000条记录；

传了startIndex，一共最多可以获取5000条， 但是这个可以分页获取

so: 不设置startIndex， 直接获取所有的记录， 目前日期范围设置为最近5天的，记录条数还足够



##### 币种设置

currency：USD



##### 语言设置

locale： zh_CN



##### 指标设置

metric： 列举出全部

```python
'AD_REQUESTS','AD_REQUESTS_COVERAGE','AD_REQUESTS_CTR','AD_REQUESTS_RPM','CLICKS','COST_PER_CLICK','EARNINGS','INDIVIDUAL_AD_IMPRESSIONS','INDIVIDUAL_AD_IMPRESSIONS_CTR','INDIVIDUAL_AD_IMPRESSIONS_RPM','MATCHED_AD_REQUESTS','MATCHED_AD_REQUESTS_CTR','MATCHED_AD_REQUESTS_RPM','PAGE_VIEWS','PAGE_VIEWS_CTR','PAGE_VIEWS_RPM',
```



##### filter的使用

https://developers.google.com/adsense/management/reporting/filtering?hl=zh-cn





```python
In [8]: result = service.accounts().list().execute()

In [9]: result
Out[9]: 
{'etag': '"J_-wNaIKqPQ6FpZptKwqdaduuK8/BNherfgtadno1jjjjBEFcw"',
 'items': [{'creation_time': '1504511042732',
   'id': 'pub-150232255235463225',
   'kind': 'adsense#account',
   'name': 'mm有限公司',
   'premium': False,
   'timezone': 'Asia/Shanghai'}],
 'kind': 'adsense#accounts'}

In [10]: result = service.accounts().adclients().list(accountId='pub-150232255235463225').execute()

In [11]: result
Out[11]: 
{'etag': '"J_-wNaIKqPQ6FpZptKSdXsdegsK8/BsefsCVtadno15iazBEFcw"',
 'items': [{'arcOptIn': True,
   'id': 'ca-app-pub-150232255235463225',
   'kind': 'adsense#adClient',
   'productCode': 'GMOB',
   'supportsReporting': True}],
 'kind': 'adsense#adClients'}
 
 In [12]: result = service.accounts().adunits().list(accountId='pub-150232255235463225', adClientId='ca-app-pub-1502
    ...: 322555229195').execute()
In [13]: result.keys()
Out[13]: dict_keys(['kind', 'etag', 'items'])

In [14]: len(result['items'])
Out[14]: 147

In [15]: result['items'][0:5]
Out[15]: 
[{'code': '1020463124',
  'id': 'ca-app-pub-150232255235463225:1020463124',
  'kind': 'adsense#adUnit',
  'name': 'Banner',
  'status': 'ACTIVE'},
 {'code': '1183405185',
  'id': 'ca-app-pub-150232255235463225:1183405185',
  'kind': 'adsense#adUnit',
  'name': 'Reward',
  'status': 'ACTIVE'},
 {'code': '1194847810',
  'id': 'ca-app-pub-150232255235463225:1194847810',
  'kind': 'adsense#adUnit',
  'name': '4K&HD page 1',
  'status': 'ACTIVE'}
```

