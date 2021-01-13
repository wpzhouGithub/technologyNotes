# [Flask Appbuild]读源码心得

1. 需要使用的类赋值给变量，不要直接在代码里面调用；这样方便后期维护修改

```python
class SecurityManager(BaseSecurityManager):
    """
        Responsible for authentication, registering security views,
        role and permission auto management

        If you want to change anything just inherit and override, then
        pass your own security manager to AppBuilder.
    """

    user_model = User
    """ Override to set your own User Model """
    role_model = Role
    """ Override to set your own Role Model """
    permission_model = Permission
    viewmenu_model = ViewMenu
    permissionview_model = PermissionView
    registeruser_model = RegisterUser
```

2. 使用三方类比较多的，可基于三方类自定义一个base类； 优点：
   - 后期如果三方类的使用方法有变或者想换一个其他的三方类使用，只要修改自定义的类即可，不用每个调用的地方都改；
   - 后期想要重写三方类的某个方法
   - 

```
add_permission_view_menu
add_permissions_menu
add_permissions_view
```



## 妙用的语法

```python
[p for p in (port, sn_port) if p is not None]
host = host or sn_host or _host

next((p for p in (port, sn_port) if p is not None), _port)
def next(iterator, default=None): 
    """
    next(iterator[, default])
    
    Return the next item from the iterator. If default is given and the iterator
    is exhausted, it is returned instead of raising StopIteration.
    """
    pass
```



## 整体思路

### 一个请求

HTTPServer监听在指定端口，接受外部请求
当一个请求到达时，它被封装成HTTPConnection对象放入消息队列
HTTPServer内部的线程池会不断的从消息队列中取出HTTPConnection对象交给WSGI网关处理
WSGI网关按照WSGI规范调用Web应用
Web应用响应发过来的请求，然后把结果会送给WSGI网关
WSGI网关又会把结果会送给HTTPServer
最终再由HTTPServer把结果写回到socket返回给客户端
Nginx、uWSGI、Flask的架构模式
Web浏览器发送HTTP请求给Nginx
Nginx转发给uWSGI
uWSGI将其按照WSGI规范进行处理
uWSGI将HTTP请求给到Flask应用进行处理
Flask应用对HTTP进行处理
比如查询数据库获取信息
将信息拼装成HTML网页
Flask产生HTTP应答，按照WSGI规范给到uWSGI
uWSGI将HTTP应答以TCP方式发给Nginx
Nginx将HTTP应答以TCP方式发给Web浏览器





### WSGI

全称 Web Server Gateway Interface，或者 Python Web Server Gateway Interface ，是为 Python 语言定义的 Web 服务器和 Web 应用程序或框架之间的一种简单而通用的接口。

从名字就可以看出来，这东西是一个Gateway，也就是网关。网关的作用就是在协议之间进行转换。WSGI 是作为 Web 服务器与 Web 应用程序或应用框架之间的一种低级别的接口，以提升可移植 Web 应用开发的共同点。



### uwsgi的启动多进程+多线程工作原理

- uwsgi是用c语言写的一个webserver，**可以启动多个进程，进程里面可以启动多个线程来服务。进程分为主进程和worker进程，worker里面可以有多个线程**。

- 代码运行过程

  1. 一开始进入main函数，启动这个就是主进程了
  2. uwsgi_setup函数（主进程）里面
     - 针对选项参数做一些处理
     - 执行环境设置，执行一些hook
     - 语言环境初始化（python）；**如果没有设置延迟加载app，则app在主进程加载；如果设置了延迟加载，则在每一个worker进程中都会加载一次**。
     - 执行插件的初始化，比较典型的如下：
       -  http gateway类型的插件，向外暴露http服务
       - python的requests类型的插件，用来服务请求的
       - tcp socket的绑定与监听 指http与work之间的通信，且它的端口是自动产生的
     - 最后uwsgi主进程fork了指定worker进程用来接收(accept)请求。虽然在setup中就fork了子进程，但是现在还没有开始accept。
  3. wsgi_run函数就是真正的开始执行了，这个函数分为俩个部分，主进程执行部分和worker进程执行部分。
     - 主进程执行部分是一个无限循环，他可以执行特定的hook以及接收信号等，总之是用来管理worker进程以及一些定时或者事件触发任务。
     - worker部分：注册信号处理函数，执行一些hook，循环接收(accept)请求。在启动woker时可以根据--threads参数指定要产生的线程个数，否则只在当前进程启动一个线程。这些线程循环接收请求并处理。

- 在worker中接收请求的函数wsgi_req_accept有一个锁：thunder_lock，这个锁用来串行化accept，防止“惊群”现象

  - 惊群现象出现在这样的情况：主进程绑定并监听socket，然后调用fork，在各个子进程进行accept。无论任何时候，只要有一个连接尝试连接，所有的子进程都将被唤醒，但只有一个会连接成功，其他的会得到一个EAGAIN的错误，浙江导致巨大的CPU资源浪费，如果在进程中使用线程，这个问题被再度放大。一个解决方法是串行化accept，在accept前防止一个锁。

    

### flask develop模式大概流程

- 主进程的调用栈：flask 对象的run函数为入口，最后的服务监听socket的selector，有请求时，启动线程，处理请求（run_wsgi）

```python
serve_forever, socketserver.py:241
serve_forever, serving.py:735
inner, serving.py:967
run_simple, serving.py:1010
run, app.py:990
<module>, run.py:4
```

- 子线程的调用栈:在子线程中调用process_request_thread，主要是`wsgi_app(self, environ, start_response)`

```python
wsgi_app, app.py:2441
__call__, app.py:2463
execute, serving.py:292
run_wsgi, serving.py:304
handle_one_request, serving.py:362
handle, server.py:418
handle, serving.py:327
__init__, socketserver.py:724
finish_request, socketserver.py:364
process_request_thread, socketserver.py:654
run, threading.py:864
_bootstrap_inner, threading.py:916
_bootstrap, threading.py:884
```

```python
    def wsgi_app(self, environ, start_response):
        """The actual WSGI application. This is not implemented in
        :meth:`__call__` so that middlewares can be applied without
        losing a reference to the app object. Instead of doing this::

            app = MyMiddleware(app)

        It's a better idea to do this instead::

            app.wsgi_app = MyMiddleware(app.wsgi_app)

        Then you still have the original application object around and
        can continue to call methods on it.

        .. versionchanged:: 0.7
            Teardown events for the request and app contexts are called
            even if an unhandled error occurs. Other events may not be
            called depending on when an error occurs during dispatch.
            See :ref:`callbacks-and-errors`.

        :param environ: A WSGI environment.
        :param start_response: A callable accepting a status code,
            a list of headers, and an optional exception context to
            start the response.
        """
        ctx = self.request_context(environ)
        error = None
        try:
            try:
                ctx.push()
                response = self.full_dispatch_request()
            except Exception as e:
                error = e
                response = self.handle_exception(e)
            except:  # noqa: B001
                error = sys.exc_info()[1]
                raise
            return response(environ, start_response)
        finally:
            if self.should_ignore_error(error):
                error = None
            ctx.auto_pop(error)
            
    def full_dispatch_request(self):
        """Dispatches the request and on top of that performs request
        pre and postprocessing as well as HTTP exception catching and
        error handling.

        .. versionadded:: 0.7
        """
        self.try_trigger_before_first_request_functions()
        try:
            request_started.send(self)
            # 此处会运行 处理请求之前的方法
            rv = self.preprocess_request()
            if rv is None:
                rv = self.dispatch_request()
        except Exception as e:
            rv = self.handle_user_exception(e)
        return self.finalize_request(rv)      
```



### flask __init__