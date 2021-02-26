# Python 的变量作用域和 LEGB 原则

理解Python的LEGB原则是理解Python命名空间的关键，而理解Python的命名空间又是理解Python中许多语法规定的关键。



## 命名空间

在 Python 程序中创建、改变或查找变量名时，都是在一个保存变量名的地方进行中，那个地方我们称之为命名空间。作用域这个术语也称之为命名空间。

**Python的命名空间是一个字典，字典内保存了变量名称与对象之间的映射关系**，因此，查找变量名就是在命名空间字典中查找**键-值对**。

python的命名空间如下：

- L-Local(function)；函数内的名字空间
- E-Enclosing function locals；外部嵌套函数的名字空间(例如closure)
- G-Global(module)；函数定义所在模块（文件）的名字空间

B-Builtin(Python)；Python内置模块的名字空间

## LEGB原则

Python有多个命名空间，因此，需要有规则来规定，按照怎样的顺序来查找命名空间，**LEGB就是用来规定命名空间查找顺序的规则**。

> LEGB规定了查找一个名称的顺序为：local-->enclosing function locals-->global-->builtin

