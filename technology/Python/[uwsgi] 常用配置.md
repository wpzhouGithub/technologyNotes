## 参数解释

- reload-on-as

当一个工作进程的虚拟内存占用超过了限制的大小，那么该进程就会被回收重用（重启）。

--reload-on-as 128
这个配置会重启所有占用虚拟内存超过128M的工作进程。当工作进程因此重启时，本次请求的响应不会受影响，返回正常结果。

- reload-on-rss

跟reload-on-as的效果类似，不过这个选项控制的是物理内存。你可以同时使用这2个选项：

uwsgi:
reload-on-as: 128
reload-on-rss: 96
这个配置会导致所有占用128M以上虚拟内存或者超过96M物理内存的工作进程重启。

当工作进程因此重启时，本次请求的响应不会受影响，返回正常结果。



- harakiri

这个选项会设置harakiri超时时间（可以看wiki首页的相关内容）。如果一个请求花费的时间超过了这个harakiri超时时间，那么这个请求都会被丢弃，并且当前处理这个请求的工作进程会被回收再利用（即重启）。

--harakiri 60
这个设置会使uwsgi丢弃所有需要60秒才能处理完成的请求。

- harakiri-verbose

When a request is killed by harakiri you will get a message in the uWSGI log. Enabling this option will print additional info (for example in Linux will be reported the current syscall)

当一个请求被harakiri杀掉以后，你将在uWSGI日志中得到一条消息。激活这个选项会打印出额外的信息（例如，在linux中会打印出当前的syscall）。

--harakiri-verbose
以上配置会开启harakiri的额外信息。



- max-requests

为每个工作进程设置请求数的上限。当一个工作进程处理的请求数达到这个值，那么该工作进程就会被回收重用（重启）。你可以使用这个选项来默默地对抗内存泄漏（尽管这类情况使用reload-on-as和reload-on-rss选项更有用）。

[uwsgi]
max-requests = 1000
上述配置设置工作进程没处理1000个请求就会被回收重用。



- http-timeout

设置内部http的socket超时时间。

http_timeout表示设置内部http的socket超时时间，如果超过这个时间没有响应就会重启该子进程



- ### [reaper](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#reaper)

  开启reaper模式。没处理一个请求，服务器就会调用waitpid(-1)来清除所有的僵尸进程。如果你在你的app中生成了子进程，结束后又成为了很多僵尸进程，那么你可以开启这个选项。但遇到这种情况你更应该修复你这种子进程的使用方式（如果可以的话）。

- ### [socket-timeout](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#socket-timeout)

  为所有的socket操作设置内部超时时间（默认4秒）。

  ```
  --socket-timeout 10
  ```

  这个配置会结束那些处于不活动状态超过10秒的连接。

- ### [disable-logging](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#disable-logging)

  不记录请求信息的日志。只记录错误以及uWSGI内部消息到日志中。

- ### [check-interval](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#check-interval)

  主进程每秒都会进行一次扫描。如果需要你可以加长这个扫描时间。不建议。

- ### [cheap](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#cheap)

  延迟启动工作进程直到第一个请求到来。当第一个请求来时就会启动预先设置个数的工作进程。

  如果没有设置工作进程数，那么最多只会启动一个工作进程。

- ### [cheaper](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#cheaper)

  一个高阶的cheap模式，这个选项将在启动的时候只分配n个工作进程，并将使用多种算法来实现适应性的进程启动。

  在启动的时候至少会分配一个工作进程，即及时设置n为0，也会在最开始启动一个工作进程。

  如果当前工作进程不足以处理收到的请求，那么就会按请求量按需启动其他的工作进程，直到工作进程达到预先设置的个数。

  如果没有设置工作进程数，那么只会在刚启动的时候启动一个工作进程，之后将不再启动其他进程。

  ```
  -cheaper <n>
  ```

- ### [idle](https://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html#idle)

  **在经过<secs>秒的不活跃状态后销毁工作进程（这时就进入了cheap模式），只会剩下主进程。如果有cheaper选项，那么就进入cheaper模式，之后收到第一个请求就会启动工作进程，启动的个数由cheaper选项里配置的n决定**

  ```
  --idle <secs>
  ```



## cheaper 的具体使用

```
# set cheaper algorithm to use, if not set default will be used
cheaper-algo = spare

# minimum number of workers to keep at all times
cheaper = 2

# number of workers to spawn at startup
cheaper-initial = 5

# maximum number of workers that can be spawned
processes = 10

# how many workers should be spawned at a time
cheaper-step = 1

idle
```



这个配置将会告诉WSGI负载之下最多运行10个worker。如果应用处于idle状态，那么uWSGI将会停止worker，但它总是会让至少2个worker在运行。使用 `cheaper-initial` ，你可以控制在启动的时候应该生成几个worker。如果你的平均负载要求比最小数量的worker还要多，那么你可以让它们立即生成，然后在负载足够低的情况下，”省省” (杀死它们)。当cheaper算法决定它需要更多的worker时，它会生成它们的 `cheaper-step` 。这在你有一个高的最大worker数的时候有用 —— 否则在突然尖峰负载的情况下，它会花费大量的时间来一个一个生成足够的worker。

以上配置的启动过程：

1. 在最初启动时，受参数cheaper-initial的控制，启动5个worker
2. 如果作业负载较低，有worker处于idle超过idle设置的秒数，将会被停止；保留cheaper个worker
3. 负载高的时候，按cheaper-step设置的个数，每次启动cheaper-step个worker，直到最大个数processes

- 当前的配置：

```shell
[uwsgi]
processes = 10
stats = 127.0.0.1:18201
socket = :8201
max-requests = 100000
log-slow = true
module = run 
callable = app
enable-threads = true
reload-on-as = 200
harakiri = 5
daemonize = /var/app/log/launcher_service/uwsgi.log
master = true
disable-logging = false
listen = 4096
py-tracebacker = /tmp/tbsocket.
cheaper = 2
idle = 100
```

预想的运行情况：

1. 启动的时候，会保持2个worker： 1、2
2. 如果worker 1卡死，那 worker 2负载太高，启动新的worker 3； 当前一个有3个worker
3. worker 1 卡死时的状态为idle，那超过100秒之后，worker 1 将会销毁
4. 当 worker 2 3 中有卡死的现象时，又会起新的worker， 然后卡死的worker会被销毁， 如此循环

真实的运行情况：

1. 启动的时候，会保持2个worker： 1、2

2. 如果worker 1卡死， worker 2继续工作，暂时还没有达到高负载

3. worker 2 运行一段时间之后，也卡死

4. 在经过idle = 100秒的不活跃状态后销毁工作进程（这时就进入了cheap模式），只会剩下主进程； 这块是所有的工作进程都不活跃超过100秒之后，销毁所有工作进程，只剩下主进程； 此处需要将idle设置的小一点，当前设置的100秒，这样会导致服务长达100秒无法提供服务

5. 等待新的连接，收到第一个请求就会启动工作进程，启动的个数由cheaper选项决定； 即worker1 2 都被重新启动了；            

   

测试环境高压并发，触发高负载状态









--heartbeat

--reload-on-exception   reload a worker when an exception is raised

 --idle                                 set idle mode (put uWSGI in cheap mode after inactivity)

--die-on-idle                          shutdown uWSGI when idle

