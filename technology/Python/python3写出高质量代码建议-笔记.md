## format 使用

### 通过类的属性填充

```
# 通过类的属性填充
class Names():
	obj = 'world'
	name = 'python'
	
print('{names.obj} in {names.name}'.format(names=Names))
#world in python

```

### 通过key填充

```
#通过key填充
print('{obj} in {name}'.format(obj='world', name='python'))
#world in python
```



## 三元符的使用

```
X if C else Y

```



## 编写 Pythonic 代码

### 要避免劣化代码，不使用不合适的变量命名

- 避免只用大小写来区分不同的对象
- 避免使用容易引起混淆的名称，包括：
  - 重复使用已经存在于上下文中的变量名来表示不同的类型
  - 误用了内建名称来表示其他含义的名称而使之在当前命名空间被屏蔽
  - 没有构建新的数据类型的情况下使用类似于 `element、list、dict` 等作为变量名
  - 使用 o（字母 O 小写的形式，容易与数值 0 混淆）、1（字母 L 小写的形式，容易与数字 1 混淆）等作为变量名



## 将常量集中在一个文件里

通过命名风格来提醒使用者该变量代表的意义为常量，如常量名所有字母大写，用下划线连接各个单词

### 常量类实现
通过自定义的类实现常量功能。在类中限制”命名全部为大写“和”值一旦绑定便不可再修改“这两个条件。下面是一种较为常见的解决办法，它通过对常量对应的值进行修改时或者命名不符合规范时抛出异常来满足以上常量的两个条件。

```
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

```
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

   

### sys.modules的使用

 sys.modules是一个全局字典，该字典是**python启动后就加载在内存中**。每当程序导入新的模块，**sys.modules都将记录这些模块**。字典sys.modules对于加载模块起到了**缓冲的作用**。当某个模块第一次导入，字典sys.modules将自动记录该模块。**当第二次再导入该模块时，python会直接到字典中查找，从而加快了程序运行的速度**。



## S9 数据交换值的时候不推荐使用中间变量

交换两个变量的值，熟悉的代码如下：

```
temp = xx = yy = temp
```

实际上，在 Python 中有 Pythonic 的实现方式，代码如下：

```
x, y = y, x
```

上面的实现方式不需要借助任何中间变量并且能够获取更好的性能（可以用 `timeit` 测试）。

之所以更优，是因为 Python 表达式计算的顺序。一般情况下 **Python 表达式的计算顺序是从左到右，但遇到表达式赋值的时候表达式右边的操作数先于左边的操作数计算**，其在内存中执行的顺序如下：

- 先计算右边的表达式 y, x，因此先在内存中创建元组（y, x），其标示符合值分别为 y、x 及其对应的值，其中 y 和 x 是在初始化时已经存在于内存中的对象。

- 计算表达式左边的值并进行赋值，元组被依次分配给左边的标示符，通过解压缩（unpacking），元组第一标识符（为 y）分配给左边第一个元素（此时为 x），元组第二个标识符（为 x）分配给第二个元素（为 y），从而达到实现 x、y 值交换的目的。

  

## for map 推导式性能对比
性能从做到右逐渐降低
推导式》map》for

尽量使用推导式，效率高、代码简洁




