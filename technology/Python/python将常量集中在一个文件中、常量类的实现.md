 @[TOC](将常量集中在一个文件里、常量类的实现、sys.modules的使用)

通过命名风格来提醒使用者该变量代表的意义为常量，如常量名所有字母大写，用下划线连接各个单词

# 常量类实现

通过自定义的类实现常量功能。在类中限制”命名全部为大写“和”值一旦绑定便不可再修改“这两个条件。下面是一种较为常见的解决办法，它通过对常量对应的值进行修改时或者命名不符合规范时抛出异常来满足以上常量的两个条件。

```python
# constant.py
class _const:
    class ConstError(TypeError):
        pass
    class ConstCaseError(ConstError):
        pass

    def __setattr__(self, name, value):
        if name in self.__dict__:
            raise self.ConstError("Can't change const.{}".format(name))
        if not name.isupper():
            raise self.ConstCaseError("const name {} is not all uppercase".format(name))
        self.__dict__[name] = value

import sys
print(__name__) #constant
sys.modules[__name__] = _const()
```

```python
# run.py
import constant

constant.NAME = 'nameOfConstant'
print(constant.NAME)
```

```
python run.py
nameOfConstant
```

run.py 中import constant的过程如下：

1. 在系统模块中查找constant 没有找到

2. 在当前模块中查找， 找到， load

3. load的过程， 先声明类名、方法名，但是方法的具体代码并没有运行，后面被调用的时候才会运行

4. **sys.modules[name] = _const() 此处手动的将constant写入sys.modules** 

5. 当constant.py 运行完之后，回到import的位置时，尝试将constant写入sys.modules； **可是此时sys.modules中已包含constant，即不会再重新写入； 这样在import constant时， 实际上获取到的是 _const()**

   

# sys.modules的使用

 sys.modules是一个全局字典，该字典是**python启动后就加载在内存中**。每当程序导入新的模块，**sys.modules都将记录这些模块**。字典sys.modules对于加载模块起到了**缓冲的作用**。当某个模块第一次导入，字典sys.modules将自动记录该模块。**当第二次再导入该模块时，python会直接到字典中查找，从而加快了程序运行的速度**。
