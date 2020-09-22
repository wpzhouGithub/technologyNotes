# 待执行项

1. 使用pep8 检查代码规范
2. 代码使用sphinx生成文档
3. 







# 待学习

1. 异常处理需要加强
2. 单元测试
3. 多进程多线程变量的共用、保存
4. AOP编程
5. 

### functools.lru_cache

### enum

元类

函数注释



需要能快速切换版本的运行环境 
现在存在着virtualenv pyenv conda anaconda等虚拟环境 



Anaconda 一个科学计算环境，Python的发行版本 ：包括了Conda 
Conda --包和虚拟环境管理工具 
virtualenv 轻量级第三方虚拟环境管理工具,没有Anaconda好用 
pyenv 　　python版本管理工具





在Python里面，我们可以先判断一个类，有没有这个函数，如果有，则获取这个函数，然后再调用。所以，我们的代码可以写成这样：

```
class A:
    def fetch_func(self, action_name):
        func= getattr(self, action_name, None)
        return func

    def execute(self, action, msg):
        func= self.fetch_func(action)
        if func is None:
            return False, "Action not found"
        return func(action, msg)
        
```



Python 的 packaging/unpackaging 机制



python  换行







