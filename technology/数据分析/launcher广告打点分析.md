 



```
`%matplotlib inline import pandas as pd from datetime import datetime import arrow import numpy as np  import matplotlib.pyplot as plt import json  plt.rcParams['figure.figsize'] = [10, 5]  headers = ["c_adId","c_ip","c_mac","c_language","c_did","c_appvc","c_appvn","c_pn","c_title","c_clickarea","log_ip","log_time","addinfo"]   parse_dates=["log_time", "c_curTime"] parser = lambda x:arrow.get(x, "YYYYMMDDHHmmss").datetime  df = pd.read_csv("c:\\workspace\\notebook\\alldata(2019.10.25).csv",                   encoding='utf-8',                  error_bad_lines =False,                  parse_dates=parse_dates)`
```

数据加载过程中，能解析出来的数据为110,751条，导出的CSV共有122819行，无法解析的数据占比为10%。因此要注意如果分析结果可能因此出现不准确的情况。

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_14-48-32.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_14-48-32.png?version=1&modificationDate=1572504512491&api=v2)



用户平均点击次数如下图所示：

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_14-50-45.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_14-50-45.png?version=1&modificationDate=1572504645064&api=v2)

上图说明，点击次数最头部的1.25%将显著拉伸平均点击次数，在这里形成一个拐点。在此之前，平均点击次数随着点击行为的纳入比例增加而线性增加。但如果仔细一点看，可以认为在85%（即平均点击次数为1.7次）之前，曲线更接近线性，而之后点击次数的增长速度略高于纳入比例的增加，直到98.75%以后，大幅攀升。

上图是取09/27~10/25间的所有数据。如果分别取10/16,10/17和10/18的数据来分析，也会得到非常接近于上图的特征（和指标）。这说明CL中用户点击行业（包括恶意点击后）是比较稳定的。

值得注意的是，09/27~10/25间所有用户点击的平均次数本应该高于单日数据，但实际上却表现出近乎一致的指标特征。这说明，我们系统中大量广告点击是由新用户完成的。即除了少数恶意点击者以外，多数正常的用户，在此期间应该只出现一次（或者其它较小的数字）。

# “异常”的一天

可以看出，从10/16日以来到10月23日，广告点击次数基本稳定（10月23日有上升）。此前（10/16）的爬坡应为发布灰度增加所致。为了便于分析，以下将全部数据分为三段，即10/15日以前（含），10/16-10/22, 10/24；其中，10/16-10/22的数据被认为是正常情况下的数据分布,记为df1016；而10/24则代表了包含异常行为的数据分布，记为df1024;



```
`df1016 = df[(df['log_time']>='2019-10-16')&(df['log_time']<='2019-10-22')] df1024 = df[(df['log_time']>='2019-10-24')&(df['log_time']<'2019-10-25')]`
```

先来看各广告位请求数的情况：

```
`g_adId_1016 = df1016.groupby(['c_adId']).describe()/len(df1016) g_adId_1024 = df1024.groupby(['c_adId']).describe()/len(df1024) g_adId = pd.merge(g_adId_1016['c_appvc']['count'], g_adId_1024['c_appvc']['count'], on='c_adId') g_adId.plot(kind='bar')`
```

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > output_6_1.png](http://portal.dolphin-x.info/download/attachments/14975220/output_6_1.png?version=1&modificationDate=1572180373913&api=v2)

黄色条是1024号各广告位点击率分布；蓝色条是正常状态下各广告位的点击率分布。

由此可以看出，在10/24那一天，尾数为2800的广告位点击率飙升，占到全天点击率的50%，是此前正常状态下的约10倍。当天其它广告位的点击分布仍为正常。

因此，接下来我们重点考察这一广告位的点击行为。



```
`df_suspicious = df1024[df1024['c_adId'] == 'ca-app-pub-4791268687937131/5976302800' ].drop('c_adId', axis=1)  len(df_suspicious)/len(df1024)`
```



```
0.5019114951557999
df_suspicious.groupby('c_ip').size().nlargest(10)
c_ip
100.77.66.161    9300
100.85.246.88      14
10.176.203.82      13
192.168.1.3         8
10.208.247.28       6
10.43.146.234       6
192.168.1.105       6
192.168.1.2         6
10.89.173.128       5
100.73.16.125       4
dtype: int64
df_suspicious.groupby('c_mac').size().nlargest(10)
c_mac
060b8135b2c0    9300
0adcb05fee77      13
94d0296c5c5f       6
eaf3c46ee5d8       6
348a7b2312f8       4
c0d3c0b15b69       4
20326c73c0c0       3
5c51819df877       3
88797e276e9c       3
986f60a5b2a5       3
dtype: int64
df_suspicious.groupby('c_title').size().nlargest(10)
c_title
50% Off* on 1st 5 Order                               9300
Google Pay (Tez) - a simple and secure payment app      30
MiChat - Chat Gratis & Bertemu dengan Orang Baru        15
Lamour Love All Over The World                          14
Fosholi - Best Agriculture App                          13
Files by Google: Clean up space on your phone           10
Snapchat                                                 8
Loan Instant Personal Loan App - CashBean                7
AJIO Flat-Knit Pullover with Asymmetric...               6
Files by Google: Bersihkan ruang di ponsel Anda          6
dtype: int64因此，有9300次点击来自同一台机器（同IP和同一MAC），其目的是为了点击一个名为50% off* on 1st 5 Order的广告。我们再看一下它的时间分布：
```



```
`df_suspicious.set_index('log_time', inplace=True, drop=True)  df_suspicious['c_mac'].groupby(pd.Grouper(freq='1h')).count().plot(kind='bar')`
```



![海豚湾 > 2019/10/27 > admob广告点击事件分析 > output_14_1.png](http://portal.dolphin-x.info/download/attachments/14975220/output_14_1.png?version=1&modificationDate=1572180391094&api=v2)

看来点击主要集中在14：00左右。我们进一步放大这一区域。经过一系列的放大和查找，最终锁定时间段为2019-10-24 14:38~2019-10-24 14:42：



```
`df_suspicious_1h = df_suspicious[(df_suspicious.index>'2019-10-24 14:37')&(df_suspicious.index<'2019-10-24 14:43')]  df_suspicious_1h['c_mac'].groupby(pd.Grouper(freq='1min')).count().plot(kind='bar')`
```



![海豚湾 > 2019/10/27 > admob广告点击事件分析 > output_16_1.png](http://portal.dolphin-x.info/download/attachments/14975220/output_16_1.png?version=1&modificationDate=1572180405388&api=v2)



# 其它（“正常”）时间的数据分析  

除了10/24号以外，其它时间的数据都是正常的吗？以下我们从10/16日起，逐日进行分析。

## 10月16日

```
`# 2019-10-16 tmp_df = df1016[(df1016['log_time']<'2019-10-17')&(df1016['log_time']>='2019-10-16')] print(tmp_df.groupby('c_mac').size().nlargest(5)) tmp_df.groupby('c_mac').size().hist(bins=200)`
```

共有7374条数据，涉及1358部设备（按mac分），排在最前面的5部设备的广告点击量分别为：

```
c_mac
608e082ada52    143
2cfdab3be631     51
74badb6dc540     43
8e39907be6df     40
580203040506     39分别统计点击次数小于3次，等于4次，5次，6-9次及10次以上的分布，得到下表：
```

|          | <=3   | 4     | 5     | 6-9   | >=10  |
| :------- | :---- | :---- | :---- | :---- | :---- |
| 点击次数 | 1068  | 92    | 54    | 86    | 58    |
| 点击占比 | 78.6% | 6.77% | 3.97% | 6.33% | 4.27% |

因此从2019/10/16的数据来看，假设一个正常的用户每天最多点击5次广告，将导致10.5%的人被限制，但超过5次以上的点击行为，很可能并不会为我们带来任何有效收益（即使用户是因为网络不好的原因重复点击，这个重试次数也够了），反而更可能是恶意点击。而如果将3次以上的点击限制住，则我们将会限制住4~5之间的另外146人，这些人当中，可能有相当一部分还是正常的用户

```
下图展示了所有点击的直方图分布。从图中可以看出，只点击一次的人约700，点击两次的约占250，点击3次的人约占150。先看几个同一用户当天点击了4次广告的，共有92人，可以看到，这里面既有点同一支广告，也有点不同广告的人：
```

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-30_12-29-57.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-30_12-29-57.png?version=1&modificationDate=1572409796902&api=v2)

（以上数据请见[1016_click_4.txt](http://portal.dolphin-x.info/download/attachments/14975220/1016_click_4.txt?version=1&modificationDate=1572409894753&api=v2)）

当天点击次数为5次的，共54人。

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-30_13-58-5.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-30_13-58-5.png?version=1&modificationDate=1572415084947&api=v2)

[数据请见（1016_click_5.txt](http://portal.dolphin-x.info/download/attachments/14975220/1016_click_5.txt?version=1&modificationDate=1572415101686&api=v2)）

点击次数超过5次的共144人，占10%。

点击次数为6次的，共28人。

点击次数为7次的，共25人。我们看看其中一个当天点击了7次广告的“用户”（mac地址6261c19f3ba8）：

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-30_10-59-40.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-30_10-59-40.png?version=1&modificationDate=1572404379791&api=v2)

这个也可能是某种薅羊毛用户，但他点击广告的时间分布分了两个时间段，一是03:47左右，另一个是03:58分左右。前一段共点击2次，后一段共点击6次。但也可能是个合理的用户，他在03:47左右看到这条广告，犹豫了一下，觉得这个广告的折扣还是比较诱人的，于是在10分钟之后，再次刷到这个广告进行了点击。

```
当天点击10次广告以上的，共58人（58/1358=4.27%）我们看看某个一天点击12次广告的“用户”（mac地址5a5d37f11ddb），它的点击时间分布如何：
```

这也许是一个正常用户，因为从他的行为时间分布来看，并没有特别不合理的地方。但是他点击的都是同一个广告，这样的多次点击，并不能给我们带来比只点击一次要多的收益。

因此从2019/10/16的数据来看，假设一个正常的用户每天最多点击5次广告，将导致10.5%的人被限制，但超过5次以上的点击行为，很可能并不会为我们带来任何有效收益（如果用户是因为网络不好的原因重复点击，这个重试次数也够了），反而更可能是恶意点击。而如果将3次以上的点击限制住，则我们将会限制住4~5之间的另外146人，这些人当中，可能有相当一部分还是正常的用户。

## 10月17日

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_11-31-11.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_11-31-11.png?version=1&modificationDate=1572492670949&api=v2)

对点击前5的用户的点击行为进行分析：

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_12-8-24.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_12-8-24.png?version=1&modificationDate=1572494903913&api=v2)

那些点击单一广告多次的，比如a6cfff67f83f这个用户，将"Great farm & city game"点击了69次，判为恶意点击应该没有疑义；但是象d4a1484d5bef这样的，全天共点击了53次，虽然分布在3支广告上，但只少的一支广告点击了6次，其它两支分别为17和30，应该也能判为恶意。

580203040506这个用户的点击行为比较奇怪，他只有两支广告点周次数较高（Find singles near you和Make the right move!),其它的广告点击次数都还算合理。不过，他一共点击了20支广告，似乎也很难相信这当中会有多少有效转化（购买行为）。

下面重点分析点击次数为10次的用户：

get_user_activity(df1017, '04946b25ccca')



![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_12-18-42.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_12-18-42.png?version=1&modificationDate=1572495521767&api=v2)

这个用户点击并不频繁，仅从时间上来看，无法判断是否为恶意点击，因为正常人类完全能完成这样的动作。但两支广告，一支点击了3次，另一支点击了7次，似乎也并不是正常行为。

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_12-22-15.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_12-22-15.png?version=1&modificationDate=1572495734996&api=v2)

这个用户的点击行为似乎比较确定为恶意点击。这应该是一个聊天应用，如果用户有意安装的话，前几次尝试应该就能够成功。如果不能成功，也应该在应用商店里重试。

在只点击了5次的用户当中，其行为更容易解释：

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_12-27-55.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_12-27-55.png?version=1&modificationDate=1572496074574&api=v2)

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_12-28-58.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_12-28-58.png?version=1&modificationDate=1572496138468&api=v2)

用户看上去在1秒中完成了多个点击，但这也有可能是操作系统把在人类看起来的一个点击，向我们的应用报告了多次而已。用户虽然点击的都是同一支广告，但IP换了，可能用户的使用场景发生了切换。

## 10月18日

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_13-59-24.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_13-59-24.png?version=1&modificationDate=1572501564649&api=v2)

这一天点击的时间分布与10月17日有不同，高峰期出现在20：00。可能与新用户来源区域有关。排名前5的用户点击的广告如下：

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_14-3-28.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_14-3-28.png?version=1&modificationDate=1572501807771&api=v2)

这里2cfdabfc5ba1当天点击了多支广告，除此之外，还在其它日期有较多点击，如下图所示：

![海豚湾 > 2019/10/27 > admob广告点击事件分析 > image2019-10-31_14-13-37.png](http://portal.dolphin-x.info/download/attachments/14975220/image2019-10-31_14-13-37.png?version=1&modificationDate=1572502416786&api=v2)

# 结论

1. 从已知的CL数据来看，每天都会有大量明显不合理的、甚至是人类无法达成的广告点击，可以认为这些点击为无效点击。
2. 98.75%的用户点击次数平均值为2.5次左右。
3. 78%的人点击次数在3次以内；大约5%的人点击次数在10次以上。这个5%的人可以认为几乎全部是恶意点击用户数，因为他们的点击次数超过了98.75%的用户平均点击次数的4倍。
4. 建议系统初始化值中用户每天的点击值设置为5次。



分析工具jupyter notebook。分析过程及代码见 [admob广告点击分析.ipynb](http://portal.dolphin-x.info/download/attachments/14975220/admob广告点击分析.ipynb?version=1&modificationDate=1572506258993&api=v2)









 