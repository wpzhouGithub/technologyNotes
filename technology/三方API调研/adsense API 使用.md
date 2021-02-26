

 @[TOC](Google AdSense API 获取报告)



## 一. 整体逻辑

1. celery task定时执行，作业包括`run_admob_new`、`run_admob_history`
   - `run_admob_new`：只获取当前的数据；运行频率：`crontab(minute='*/10')`
   - `run_admob_history`：获取最近5天的数据，用于规避admob控制台历史数据的波动；运行频率：`crontab(minute='8', hour='*/2')`
2. 

3. 目前有3个不同的账号（accountId），每个账号下分应用（AD_CLIENT_ID）
4. 不同账号下的应用有交集，在获取数据之后，需要将相同应用的数据归类； 就需要维护一个AD_CLIENT_ID 与应用的映射表
5. 各个账号需要放在不同的IP下运行



## 二. token 的验证

1. console 生成json 格式的token，第一次认证的时候，需要手动打开返回的url，同意一系列的用户政策之后，认证通过

   - token有两种：web和installed

   最初用的web 的，但是打开验证url 的时候，400错误

   installed 的，打开返回的url，同意一系列的用户政策之后，认证通过 

2. 认证之后，会在当前目录生成adsense.dat，下次验证的时候，直接读取该文件

3. credit_path的设置，因为有多个账号token的存在，想把各个账号用文件夹分开，所以把client_secrets.json、adsense.dat都放到了各账号的文件夹下

```python
service, flags = sample_tools.init(
    ['--noauth_local_webserver'], 'admob', 'v1.4', __doc__, credit_path,
    scope='https://www.googleapis.com/auth/admob.readonly')

```

4. 问题来了

   - warning : adsense.dat 找不到

   - 提示url，需要重新手动验证

   - 猜测应该是adsense.dat 读取失败导致的，可是adsense.dat 明明是放在credit_path目录下的

   - **读源码得知，只会在当前目录查找adsense.dat；即并没有去credit_path指定的文件夹读取**

   - 在运行代码之前，直接cd credit_path； 这样就万事大吉了




## 三. API

官网连接： https://developers.google.com/admob/api

- 需要注意的点

  1. admob api在不filter的情况下，会返回多个平台的收入，比如admob、Facebook； 这个主要是因为我们的client app 使用了admob的open bidding

     ```python
     # Set dimension filters.
     # just admob network
     dimension_filters = {
         'dimension': 'AD_SOURCE',
         'matches_any': {
             'values': ['5450213213286189855']
         }
     }
     ```

  2. 时区默认使用用户在admob控制台设置的报告的时区，目前都设置的utc+8

  3. 运行API时，需要切到对应账号的token目录下

