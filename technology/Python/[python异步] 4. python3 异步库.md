# python异步库asyncio、uvloop

asyncio 是Python 标准库里的一个异步 I/O 框架。

 [uvloop](https://github.com/MagicStack/uvloop) : 是 asyncio 默认事件循环的一个代替品，实现的功能完整，且即插即用。uvloop 是用 Cython 写的，建于 [libuv](http://libuv.org/) 之上。

uvloop 可以使 asyncio 更快。事实上，它至少比 nodejs、gevent 和其他 Python 异步框架要快 **两倍** 。基于 uvloop 的 asyncio 的速度几乎接近了 Go 程序的速度。



在你的 asyncio 代码中使用 uvloop 非常简单：

```
import asyncio
import uvloop
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())

```

上面的代码片段让 `asyncio.get_event_loop()` 返回一个 uvloop 的实例。

你还可以显式的创建一个uvloop实例，通过调用`uvloop.new_event_loop()。`



**综上，asyncio 是python3标准库中的一个异步I/O库；uvloop是用Cython写的，基于libuv（Unix实现epoll io模型的系统库）的，实现了事件循环的功能，是asyncio 默认事件循环的一个代替品； uvloop 可以使 asyncio 更快。**

