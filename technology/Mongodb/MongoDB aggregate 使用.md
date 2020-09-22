# MongoDB aggregate 使用

### mod 整除的使用

https://docs.mongodb.com/manual/reference/operator/aggregation/mod/#exp._S_mod

![1576148432435](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1576148432435.png)

### limit 使用

https://docs.mongodb.com/manual/reference/operator/aggregation/limit/



![1576148553706](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1576148553706.png)



### skip 使用

https://docs.mongodb.com/manual/reference/operator/aggregation/skip/

![1576148655965](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1576148655965.png)



### sort 使用

https://docs.mongodb.com/manual/reference/operator/aggregation/sort/

![1576148728785](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1576148728785.png)



### match 顾虑

https://docs.mongodb.com/manual/reference/operator/aggregation/match/

![1576148876576](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1576148876576.png)

### project 输出指定字段

https://docs.mongodb.com/manual/reference/operator/aggregation/project/

![1576149096686](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1576149096686.png)



### example

```python
from pymongo import MongoClient

# 多线程在实例化MongoClient时，要加上connect = False, pymongo不是进程安全的
mongo1 = MongoClient(MONGODB_CONFIG["anthcraft"], connect=False)
anthcraft = mongo1["anthcraft"]

    fields = {
        'title': 1,
        'is_free': 1,
        'thumbnail': 1,
        'layer1': 1,
        'layer2': 1,
        'layer3': 1,
        'createTime': 1,
        'updateTime': 1,
    }
    cond = {'apkType': apkType, 'status': 1}
    number = 3
    skip = 10
    pageSize = 20
    
    table = anthcraft.wallpaper_4d
    
    
    fields_tmp = fields.copy()
    #order
    fields_tmp['num1'] = {'$floor': {'$divide': ['$order', 1000]}}
    #id
    fields_tmp['num2'] = {'$mod':
        [
            {'$add': ['$_id', number]},
            number * 10
        ]
    }
    #updateTime
    fields_tmp['num3'] = '$updateTime'

    pipeline = [
        {'$match': cond},
        {'$project': fields_tmp},
        {'$sort': {'num3': 1, 'num2': 1, 'num1': 1}},
        {'$skip': skip},
        {'$limit': pageSize},
        {'$project': fields},
    ]
    rs = table.aggregate(pipeline)
    
```

注意： 

**对于sort 的使用，排序字段的顺序是按着key字符串排序的**

之所以用num3、num2、num1 就是为了排序按指定的顺序

