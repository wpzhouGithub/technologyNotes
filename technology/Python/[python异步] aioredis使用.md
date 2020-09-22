## 一、连接池使用

因为是异步实现，在web并发时会同时有大量的连接； 所以不建议使用连接，建议直接用连接池

```python
import aioredis

async def sample_pool():
    # connect pool, return ConnectionsPool；执行命令时需要调用execute
    pool = await aioredis.create_pool('redis://localhost')
    val = await pool.execute('get', 'my-key')
    awit pool.execute('set', 'my-key', 'value')
    
# Create Redis client bound to connections pool.
async def pool_of_connections():
   # Redis client (result of commands_factory call)，可以直接运行命令；
   redis = await aioredis.create_redis_pool(
      'redis://localhost')
   val = await redis.get('my-key')

   # we can also use pub/sub as underlying pool
   #  has several free connections:
   ch1, ch2 = await redis.subscribe('chan:1', 'chan:2')
   # publish using free connection
   await redis.publish('chan:1', 'Hello')
   await ch1.get()    
```



![1574912544550](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1574912544550.png)

![1574912495503](C:\Users\wpzhou\AppData\Roaming\Typora\typora-user-images\1574912495503.png)
