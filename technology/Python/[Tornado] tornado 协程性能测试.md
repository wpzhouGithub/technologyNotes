# Tornado  协程性能测试

- 环境

  - 双核
  - python3
  - tornado 6.0.3
  - 进程数 2

- 接口1

  ```python
  class TestHandler(BaseRequestHandler):
      # @exception
      # @performance
      async def get(self):
          return self.write('ok')
  ```

- ab 测试返回

  ```shell
  Concurrency Level:      1000
  Time taken for tests:   7.117 seconds
  Complete requests:      10000
  Failed requests:        0
  Total transferred:      3030000 bytes
  HTML transferred:       20000 bytes
  Requests per second:    1405.17 [#/sec] (mean)
  Time per request:       711.659 [ms] (mean)
  Time per request:       0.712 [ms] (mean, across all concurrent requests)
  Transfer rate:          415.79 [Kbytes/sec] received
  
  Connection Times (ms)
                min  mean[+/-sd] median   max
  Connect:        0  330 730.0      1    3007
  Processing:    37  222 231.5    184    2108
  Waiting:       36  222 231.5    184    2108
  Total:         38  552 789.5    209    3445
  ```

  

- 服务能够支持的用户请求推测
  - 假设QPS=1400
  - 那每分钟能支持的请求数1400 * 60 =84000
  - 因为是双核，2个进程，所以1个进程的QPS 为1405/2=702
  - 线上2台2核机器，那么QPS=702 * 2 * 2 = 2808
  - 所以线上1分钟能够处理的请求数为2808 * 60 = 17万

