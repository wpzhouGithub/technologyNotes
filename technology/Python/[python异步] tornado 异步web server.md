- [x] 异步库： mongo 
  - [x] motor
  
  - [x] https://github.com/mongodb/motor
  
  - [x] https://motor.readthedocs.io/en/latest/examples/index.html
  
  - [x] test theme mongo 升级从2.6 到3.0+
  
  - [x] motor 2.0
  
    - [x] python3.7+
  
    - [x] cpython3.4+
  
    - [x] pymongo3.7+
  
    - [x] mongodb3.0+  线上🆗，theme test 需要升级
  
    - [x] tornado4.0+
  
- [ ] 异步库：redis
  - [x] aioredis
  - [x] https://github.com/aio-libs/aioredis
  - [x] command 使用
  - [ ] **验证redis 连接池的connection在execute命令之后有没有释放**
  - [ ] json 压缩存储
- [x] 异步库：mysql
  - [x] aiomysql
  - [ ] https://github.com/aio-libs/aiomysql
- [x] 异步库： log
  - [x] aiologstash
  - [x] https://github.com/aio-libs/aiologstash



- [x] 框架的目录设计
- [ ] tornado 使用uvloop
- [x] tornado.process.fork_processes 运行之后是不是就已经fork出n个process了，后面运行的语句是否都是在子进程运行的
  1.  是的， fork_processes语句之后的代码只在子进程运行
  2. IOLoops  在fork_processes之后才会运行
  3. 子进程死亡之后会自动重启，直到max_restarts （defaults to 100）
- [x] 数据库模块的设计
  - [x] 异步的实现
  - [x] 初始化的时机
  - [x] 连接池 异步之后同时运行的查询数会大量增加
- [x] log
- [x] 配置文件读取





异步，未使用redis 缓存时，qps大概在400



pip3 install aioredis aiomysql motor aiologger



性能测试：

1. 未使用uvloop，process=1

   ```shell
   ab -c 10000 -n 10000 http://172.16.77.13:8888/client/v3/wallpaper_4d/wallpaperList.json?pageSize=20
   Concurrency Level:      10000
   Time taken for tests:   22.251 seconds
   Complete requests:      10000
   Failed requests:        0
   Total transferred:      3370000 bytes
   HTML transferred:       350000 bytes
   Requests per second:    449.42 [#/sec] (mean)
   Time per request:       22251.084 [ms] (mean)
   Time per request:       2.225 [ms] (mean, across all concurrent requests)
   Transfer rate:          147.90 [Kbytes/sec] received
   
   ```

2. 未使用uvloop，process=4

   ```
   ab -c 10000 -n 10000 http://172.16.77.13:8888/client/v3/wallpaper_4d/wallpaperList.json?pageSize=20
   Concurrency Level:      10000
   Time taken for tests:   24.067 seconds
   Complete requests:      10000
   Failed requests:        0
   Total transferred:      3370000 bytes
   HTML transferred:       350000 bytes
   Requests per second:    415.51 [#/sec] (mean)
   Time per request:       24066.898 [ms] (mean)
   Time per request:       2.407 [ms] (mean, across all concurrent requests)
   Transfer rate:          136.74 [Kbytes/sec] received
   
   ```

   

3. 

   

   