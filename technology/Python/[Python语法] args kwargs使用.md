#  *args  **kwargs使用

[TOC]
## 写法

- 其实并不是必须写成`*args` 和`**kwargs`。 **只有变量前面的 `*`(星号)才是必须的**. 你也可以写成`*var` 和`**vars`. 而写成`*args` 和`**kwargs`只是一个通俗的命名约定

## 用途

- 主要将不定量的参数传递给函数，即当不确定函数有多少个参数的时候使用
- `*args` 是用来发送一个非键值对的可变数量的参数列表给一个函数.
- `**kwargs` 允许你将不定长度的**键值对**, 作为参数传递给一个函数。 如果你想要在一个函数里处理**带名字的参数**, 你应该使用`**kwargs`。


## 标准参数与`*args、**kwargs`在使用时的顺序

那么如果你想在函数里同时使用所有这三种参数， 顺序是这样的：

```
some_func(fargs, *args, **kwargs)
```

